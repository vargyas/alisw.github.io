---
title: Pull Request Builders
layout: page
categories: infrastructure
---

The ALICE Github PR builder is implemented as a set of independent
agents which looks at a portion of the phase space of the PRs available
for testing. This means that if there are 4 builders deployed each one
of them will build on average 1/4th of the PRs. This avoids the need
for a centralised scheduling service. Moreover, since the process of
checking a PR is not done once a given PR is built, but only when it's
merged, builders keep retrying to build even PR which were previously
successfully built. This is because the boundary condition of the PR
checking could be different (e.g. merge conflict might happen) and
therefore the test of PR is considered done only when the PR is merged.

# Deploying the builders

Deploying the builders is done via the Apache Aurora instance. You can
find instructions on how to set it up [here](infrastructure-apache).
Because of the way the system is partitioned, we can ensure fully
covered scale up / scale down operations in the following way. Say that
we want to go from 4 workers to 8, one can start the new 4 workers by
doing:

    # Start the new 4 workers by doing.
    aurora update start build/mesosci/devel/aliphysics_github_ci/4-7 aurora/continuos-integration.aurora
    # Those workers do the same job as worker 2 and 3 in the previous configuration.
    # Once they are up and running, we can therefore safely redeploy 2 and 3.
    aurora update start build/mesosci/devel/aliphysics_github_ci/2-3 aurora/continuos-integration.aurora
    # The new 2 and 3 will do the job of the old 1, which we can now redeploy
    aurora update start build/mesosci/devel/aliphysics_github_ci/1 aurora/continuos-integration.aurora
    # Finally we restart 0
    aurora update start build/mesosci/devel/aliphysics_github_ci/0 aurora/continuos-integration.aurora


# Monitoring the builders

Builders are monitored in Monalisa. In particular you can use aliendb9
and look at the `github-pr-checker/github.com/github-api-calls` metric
to know how many API calls are being done by the system.

