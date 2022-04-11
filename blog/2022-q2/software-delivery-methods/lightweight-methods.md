

The v-model expanded on this shift in thinking by changing the way organizations thought about feedback; replacing the feedback between sequential stages with a layered apporach.













A more evolutionary approach had begun to emerge, where software was written quickly in a modern language and user feedback on the system directed it's evolution. This approach was, at times, a little too similar to software that was delivered with no model at all. There was some hope that a transformational model might solve the problem by providing automatic code generation based on a set of specifications, as this would mean the code would no longer need to be maintained; only the set of specifications.




The most influential heavyweight methods include Waterfall, Spiral Model, and V-Model.

Model = order of phases and criteria for completion of a step and entrance into the next.
Method = how to navigate each phase



Herbert D. Benington

- In the 50s and 60s programmers designed, tested, installed, and maintained their program!
- "documentation of the system is an immense, expensive job" but it was necessary as the documentation was how the knowledge was shared 


- As if he were looking at the current state of software development, Benington said; "there remains a tremendous range and ability among computer programmers to do different jobs. Some are good gem-cutters for any kind of stone. Some can play very special roles - for example, where fastidious approaches are needed. Some are brilliant and articulate conceptualizers and leaders. Some should not be allowed near a computer. We must learn to recognize these types, to use them in their right place, and to set higher standards for not using people even though the market seems insatiable."
- In what is a fantastically prescient statement, Benington discussed decentralized programs, which divided into "a dozen or so parts" with communication between each part using blocks of data on magnetic tape or punch cards. He said that these blocks could be defined with relative ease to considerably simplify the design problem. "After the blocks have been documented, groups of programmers can be assigned to each part with the assurance that little communication between these programmers will be necessary..." which is the fifties version of microservices solving Conway's law.
- "It is debatable whether a program of 100,000 instructions can ever be thoroughly tested - that is, whether the program can be shown to satisfy its specifications under all operating conditions." He suggested that we "must accept the fact that testing will be sampling only." He also added that "program testing effort is seldom adequate". He also said that the component tests should be performed using simulated inputs and recorded outputs as this would make the tests reproducible and remove the question of whether an error discovered during the tests could have been introduced by a human entering the wrong inputs.

## What changed?

- Higher-order programming languages
- Being able to program using interactive consoles
- Way more companies started writing software... not just military and space industries

## Waterfall

For the purposes of the Waterfall method, we'll use the definition from Winston Royce's "Managing the Development of Large Software Systems", which was written in 1970.

**Describe Waterfall**

### What it got right

Winston Royce started by describing a two-stage software delivery method that he deemed sufficient for most small software development projects. It involved analysis and coding.

He went on to describe a common seven-phase process of software delivery and explained that the process was prone to failure due to a lack of feedback cycles and the delay between the formation of the original requirement and the testing meant that it was highly likely a large amount of rework would be needed.

"I believe in this concept," he said. "But the implementation described above is risky and invited failure."

He then suggested an iterative interaction between various phases but also acknowledged that the feedback is not constrained to successive steps.

In fact, many of the observations in "managing the development of large software systems" represent a foreshadowing of the lightweight methods movement in the 1990s. So, what did Royce get wrong?

### What it got wrong

There are really two key failures in Waterfall, the first is that Royce correctly predicted many of the critical failures in the traditional phased approach and explains them in the paper. Despite having worked out all of the issues, Royce doesn't quite put his finger on the ultimate solution to all the drawbacks he perceived: Work in smaller batches.

All evidence suggests that the bigger you make a project, the more likely it is to fail. You can multiply the budget by the planned duration and get a reasonable failure indicator from the resulting number.

The attempts in Waterfall to avert disaster add overheads without meaningfully reducing the risk at all. In fact, practitioners often found themselves in the vicious circle of slowing down delivery to try and improve quality, which is proven to have the opposite effect.

The second key failure was that Waterfall had a skewed rationality to success ratio. It *sounded* very plausible and gained wide adoption without meaningfully solving the problems that Royce so intuitively predicted.

Waterfall found its way into many traditional methods and maturity models with organizations adopting damaging practices to get certified against required delivery standards.

### Interesting ideas

Most of the key ideas in waterfall are found in Royce's own findings. The delay in getting feedback, the increase of risk in line with project size, and the added overheads of attempting to add process to control risk were all astute observations. He didn't quite get to findings such as targeting fast feedback loops or working in small batches, but he came close.

There was a principle in Waterfall called "do it twice" that suggested that a completely novel idea should be implemented once *in miniature* as a practice run and then again as a proper product. Royce suggested a one-third split for practice with two-thirds remaining for the real thing. This idea has resurfaced in other methods as a prototype, MVP, or spike that is intended to remove some uncertainties and surface any unforeseen problems before you get too heavily invested in a particular pathway.

There was also a very clear call in Waterfall to involve the customer. While there was a more contractual and commitment-based intent to this statement, customer involvement remains a key aspect of many modern software methods - just with a more collaborative nature of engagement.

## Spiral Model

In 1988, Barry Boehm described "A Spiral Model of Software Development and Enhancement" that arranged the phases of software delivery into a repeating spiral, where each journey incrementally build on the previous one.

### What it got right

### What it got wrong

### Interesting ideas

## V-Model

The V-Model is attributed to Keven Forsberg and Harold Mooz from their 1991 paper "The Relationship of System Engineering to the Project Cycle". They originally referred to it as the "Vee" Model. Interestingly, they include a version of the Watefall process that Winston Royce used to critise the view that feedback was a concern of successive steps, which perhaps contributes to the misunderstanding of Waterfall but certainly supports their alternative, which arranges the steps into a V-shape and has more verbose sounding steps.

**Describe V-Model**

### What it got right

The V-Model extended Waterfall by acknowledging Royce's own observation that feedback doesn't occur between successive steps. The shape of the V-Model reflects the more concentric-circle nature of feedback, which means the feedback on earlier steps typically arrives later in the process.


### What it got wrong

Although arranged into a v-shape, the process is really linear, rather iterative. As with Waterfall, the test phase would often be compressed due to previous stages taking longer than expected. The V-Model was highly document-driven, with formal user requirement documents, software specifications, architecture descriptions, module designs, unit test plans, integration test plans, system test plans, and user acceptance test plans all listed amongst the required documentation.

### Interesting ideas



## References

Production of Large Computer Programs. Herbert D. Benington (1956) re-published in 1983 with an additional foreword from Benington.