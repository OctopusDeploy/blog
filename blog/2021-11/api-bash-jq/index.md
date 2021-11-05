---
title: Using the Octopus API with Bash and jq
description: Learn how to work with the Octopus API using Bash and jq
author: shawn.sesna@octopus.com
visibility: private 
published: 2021-11-29-1400
metaImage: blogimage-usingtheoctopusapiwithbashandjql.png
bannerImage: blogimage-usingtheoctopusapiwithbashandjql.png
bannerImageAlt: Arrow shaped like a sideways horseshoe, with the middle section highlighted containing a square for curl and a square for jq 
isFeatured: false
tags:
 - 
---

Octopus Deploy is written API first, meaning anything you can do in the User Interface (UI), you can do with an API call.  When interacting with the API, I've typically used PowerShell as it has the built-in ability to convert JSON into PowerShell objects making it super easy to work with JSON.  However, not all Octopus customers use PowerShell and need *nix based solutions using Bash.  In this post, I'll demonstrate how to use Bash with the Octopus API.

## jq
The Octopus Deploy API returns data in JSON format.  Bash, however, doesn't have a built-in way to work with JSON data and treats it as strings.  While Bash has some sophisticated string manipulation functionality, it is still quite difficult to work with JSON data.  To combat this problem, the Linux community developed a powerfull command-line utility to parse JSON called jq.  Jq has become the go-to utility for working with JSON data and can easily be installed by using a package manager such as `apt` or `yum`.

## curl
Wget and cURL are the two most commonly used methods for making web requests when using Bash.  With so many distributions of Linux available, it's impossible to tell which or either of these utlities are installed.  The examples contained within this post all use the cURL utility.  Like jq, cURL can be installed using a package manager.

## Working with paginated data
Many of the APIs in Octopus return data in a paginated format.  For these API calls, you have the ability to provide querstring parameters such as `skip` and `take` to manipulate the results of the call (see https://[YourServer]/api to list the API methods and a list of querystring paramters for each).  There are times you may want to return all results for a given call such as retrieving all projects.  For things like this, I wrote a Bash function that recursively calls the API until all results have been returned

:::info
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

Each call to the API returns an `Items` array.  This results in the returned JSON having multiple JSON arrays within it.  The `if` statement checks to see if muliple arrays were returned by testing the string for `][` and replacing it with `,` if it is found.  This makes the JSON string a single array and easier to work with.  With all of the project data returned, you can use more jq commands to retrieve elements such as the ProjectId and do something with it.

```bash
# Iterate over returned items
arr=( $(echo "$projects" | jq -r '.[].Id') )

for projectId in "${arr[@]}"
do
    echo "$projectId"
done
```

## POSTing to the API with Bash
So far, the examples have demonstrated how to make GET requests to the API.  While this is useful, it's really only part of how you'll be interacting with the API.  The following example creates a JSON document that will be used to POST to the `interruptions` API

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
This post demonstrated how to interact with the Octopus API using Bash, cURL, and jq.  The [Octopus API Examples](https://octopus.com/docs/octopus-rest-api/examples) page contains more examples of using the API with PowerShell, C#, Python and Go.

Happy scripting!