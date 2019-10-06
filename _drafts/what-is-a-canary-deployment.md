What's a canary deployment?

Is performinng a deployment without disrupting active users.

How does it work?
It is basically an incremental deployment. 
First the build is pushed to a small set of servers and only a small set of users are directed to them. If it goes well, then the pool of servers where the build is pushed is increased as well as the number of users. If that second level goes well, then the build is pushed to all servers and allowed to all users. 

How can it be accomplished?
By using user-based load balancing rules. 

Image

One way of implementation is by removing the first set of servers from the load balancer pool and upgrade them and reinserting for testing into the LB pool before going to the next phase. Of course if the LB is programatically then you don't need to do above by just inseting some logic into the LB and you are ready to go. 