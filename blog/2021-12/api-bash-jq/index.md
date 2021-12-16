---
title: Using the Octopus API with Bash and jq
description: Learn how to work with the Octopus API using Bash and jq.
author: shawn.sesna@octopus.com
visibility: public 
published: 2021-12-06-1400
metaImage: blogimage-usingtheoctopusapiwithbashandjql.png
bannerImage: blogimage-usingtheoctopusapiwithbashandjql.png
bannerImageAlt: An arrow turning around represents an API with a cut out section of the arrow showing curl and jq inside the API 
isFeatured: false
tags:
 - DevOps
---

Octopus Deploy is written API-first, meaning anything you can do in the user interface (UI), you can do with an API call.  When I interact with the API, I use PowerShell because it has the built-in ability to convert JSON into PowerShell objects, making it easy to work with JSON. However, not all Octopus customers use PowerShell, and some need *nix-based solutions using Bash.  

In this post, I demonstrate how to use Bash with the Octopus API.

## jq
The Octopus Deploy API returns data in JSON format.  Bash, however, doesn't offer a built-in way to work with JSON data and treats JSON as strings.  While Bash has some sophisticated string manipulation functionality, it's still difficult to work with JSON data.  

To combat this problem, the Linux community developed a powerful command-line utility to parse JSON called jq.  Jq has become the go-to utility for working with JSON data and can be easily installed by using a package manager, such as APT or YUM.

## curl
Wget and cURL are the two most used methods for making web requests using Bash.  With so many distributions of Linux available, you can't tell which of these utilities is installed.  The examples in this post use the cURL utility.  Like jq, cURL can be installed using a package manager.

## Working with paginated data
Many of the APIs in Octopus return data in a paginated format.  For these API calls, you can provide querystring parameters such as `skip` and `take` to manipulate the results of the call. See https://[YourServer]/api for a list of the API methods and a list of querystring parameters for each.

There are times you want to return all results for a given call, for example when you retrieve all projects.  For situations like this, I wrote a Bash function that recursively calls the API until all results are returned.

:::hint
While this section of the post talks about paginated data, the `Get-OctopusItems` function also works with non-paginated API calls.
:::

```bash
Get-OctopusItems () {
    octopusUri=$1
    apiKey=$2
    skipCount=$3

    header="X-Octopus-Apikey: $apiKey"
    items=()
    skipItemQuerystring=""

    # Adjust querysting accordingly
    if [[ "$octopusUri" == *"?"* ]]; then
        skipItemQuerystring="&skip="
    else
        skipItemQuerystring="?skip="
    fi

    # Append the amount to skip
    skipItemQuerystring+="$skipCount"

    # Get results
    resultSet=$(curl -H "$header" -H "Accept: application/json" -H "Content-Type: application/json" "$octopusUri$skipItemQuerystring")

    # Check to see if items is present
    items=$(echo "$resultSet" | jq .Items)

    if [[ ! -z "$items" ]]; then

        # Check to see if results are bigger than page count
        itemsPerPage=$(echo "$resultSet" | jq -r .ItemsPerPage)
        totalResults=$(echo "$resultSet" | jq -r .TotalResults)
        itemCount=$(echo "$(($totalResults - $skipCount))")

        if [[ "$itemCount" -gt "$itemsPerPage" ]]; then

            # Increment skip count
            skipCount=$(echo "$(($itemsPerPage + $skipCount))")

            # Recursively call
            items+=$(Get-OctopusItems "$octopusUri" "$apiKey" "$skipCount")
        fi
    else
        echo "$items"
    fi

    echo "$items"
}

OctopusUrl="https://yourserverurl"
spaceId="Spaces-1"
ApiKey="API-YourAPIKey"

# Get all the projects
projects=$(Get-OctopusItems "$OctopusUrl/api/$spaceId/projects" "$ApiKey" 0)

if [[ "$projects" == *"]["* ]]; then
    # Replace characters to make one contiguous array
    projects=$(echo "${projects//][/,}")
fi
```

Each call to the API returns an items array.  The JSON that's returned then has multiple JSON arrays in it.  

The `if` statement checks if many arrays are returned by testing the string for `][` and replacing it with `,` if found.  This makes the JSON string a single array, which is easier to work with.  

After all of the project data is returned, you can use more jq commands to retrieve elements such as the ProjectId and retrieve project specific data like the deployment process, runbooks, or variables.

```bash
# Iterate over returned items
arr=( $(echo "$projects" | jq -r '.[].Id') )

for projectId in "${arr[@]}"
do
    echo "$projectId"
done
```

## Posting to the API with Bash
The examples above demonstrate how to make GET requests to the API.  While useful, these requests are only part of how you interact with the API. The following example creates a JSON document to POST to the interruptions API:

```bash
automaticResponseReasonNotes="Because I said so!"
automaticResponseManualInterventionResponseType="Proceed"

# Create JSON document
jsonBody=$(jq -n \
        --arg notes "$automaticResponseReasonNotes" \
        --arg result "$automaticResponseManualInterventionResponseType" \
        '{"Notes": $notes, "Result": $result}' )

# Submit response
curl -H "$header" -H 'Content-Type: application/json' -X POST -d "$jsonBody" "$automaticResponseOctopusUrl/api/$spaceId/interruptions/$manualInterventionId/submit" 
```

## Conclusion
This post demonstrates how to interact with the Octopus API using Bash, cURL, and jq.  The [Octopus API examples](https://octopus.com/docs/octopus-rest-api/examples) page contains more examples of using the API with PowerShell, C#, Python, and Go.

Happy deployments!
