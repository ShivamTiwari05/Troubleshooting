***production is down***

Prod is down. What do you check first?

Here's how I would approach SRE troubleshooting:

Step 1: Assess impact

One user or many?

One service or system-wide?

This determines the next set of actions

Step 2: Reproduce the issue

Can I see/reproduce it for myself?

Others do not see what you see

Helps cut noise.

Step 3: Start investigating from one side of stack Which side to start from comes from experience!

If Infra first:

Is the node/container healthy?

Any spikes in CPU, memory, disk?

Step 4: Check platform dependencies

DNS resolving? DB connection pool? Msg Q stuck?

Check 3rd party integrations

Step 5: App-level checks

Recent code deployments?

Unhandled exceptions in logs?

Rate limits hit?

Step 6: May the Force be with you!

Debugging is less about tools; more about thinking in layers.