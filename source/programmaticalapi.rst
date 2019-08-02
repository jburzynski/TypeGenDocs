==================
Programmatical API
==================

Programmatical API has been removed in version 2.4.0.

The main reason behind removing it is the fact that more and more logic needs to be performed on the CLI level as new TypeGen features are being implemented. In practice it means that using the programmatical API alone (without the CLI) becomes progressively more difficult and it came to a point at which it's easier to run TypeGen CLI as a separate process from the code level rather than to use the programmatical API.

The secondary reason is that the programmatical API is not very widely used, and maintaining it takes effort and complicates the codebase, so taking everything into consideration I've decided it's best to remove it.

In case someone would like to see this functionality brought back, it can be done fairly easy, so if you'd like to see the Programmatical API back, you can either raise an issue on github or e-mail me at jburzyn gmail.