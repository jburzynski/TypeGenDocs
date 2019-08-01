==================
Programmatical API
==================

Programmatical API has been removed in version 2.4.0.

The main reason behind removing it is the fact that more and more logic needs to be performed on the CLI level as new TypeGen features are being implemented. It effectively means that using the programmatical API alone (without the CLI) becomes progressively more difficult and it came to a point at which it's easier to run TypeGen CLI as a separate process from the code level rather than to use the programmatical API.

The secondary reason is that it doesn't seem to be very popular among the users, and maintaining it takes effort and complicates the codebase, so taking everything into consideration I've decided it's best to remove it.

In case there is a substantial group of people that would like to see this functionality brought back, it can be done fairly easy, so if you'd like to see the Programmatical API back, you can either raise an issue on github or e-mail me at jburzyn gmail.