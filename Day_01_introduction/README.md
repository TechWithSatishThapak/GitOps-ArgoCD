# Day_01: Introduction to GitOps and Argo CD

## What You Will Learn

By the end of this chapter, you should be able to:

-   Explain GitOps in simple terms.
-   Understand the four main GitOps principles.
-   Describe how Argo CD helps manage Kubernetes applications.
-   Understand the difference between traditional deployment pipelines
    and GitOps.

------------------------------------------------------------------------

## 1.1 What Is GitOps?

GitOps is a way to manage and deploy applications, infrastructure, and
configuration using Git.

The main idea is simple: **Git stores the desired state of your
system.**

For example, imagine you want to run three copies of an application in
Kubernetes. You describe that requirement in a Kubernetes YAML file and
store the file in a Git repository.

A GitOps tool, such as Argo CD, checks the files in Git and compares
them with what is running in the Kubernetes cluster. If the cluster does
not match the configuration in Git, Argo CD can bring it back in line.

This means you do not need to make every deployment change manually.

### A Simple Analogy

Think of Git as a recipe book:

-   **Git** is the recipe book. It contains the instructions for how the
    application should run.
-   **Kubernetes** is the kitchen where the application runs.
-   **Argo CD** is the chef. It checks the recipe and makes sure the
    kitchen follows it.

If the recipe changes, Argo CD notices the change and can update the
application.

> **Remember:** Git describes what you want. Argo CD helps make the
> Kubernetes cluster match that desired state.

------------------------------------------------------------------------

## 1.2 What Is Argo CD?

Argo CD is a GitOps continuous delivery tool for Kubernetes.

It connects a Git repository to a Kubernetes cluster and monitors both:

1.  It reads the application configuration from Git.
2.  It checks the resources currently running in Kubernetes.
3.  It compares the live state with the desired state stored in Git.
4.  It reports differences and, when configured to do so, applies the
    required changes.

Argo CD supports common Kubernetes configuration tools, including:

-   Kubernetes YAML manifests
-   Helm charts
-   Kustomize

### Important Argo CD Terms

  -----------------------------------------------------------------------
  Term                                Simple meaning
  ----------------------------------- -----------------------------------
  **Desired State**                   How you want your application to
                                      look, as described in Git.

  **Live State**                      What is actually running in the
                                      Kubernetes cluster.

  **Sync**                            Applying the configuration from Git
                                      to the cluster.

  **Drift**                           A difference between the desired
                                      state and the live state.

  **Self-Heal**                       Automatically correcting certain
                                      live changes that do not match Git,
                                      when enabled.

  **Application**                     An Argo CD resource that connects a
                                      Git source to a destination cluster
                                      and namespace.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 1.3 The Four GitOps Principles

GitOps is commonly explained through four main principles.

### 1. Declarative

Describe the result you want, not every command needed to create it.

For example, a Kubernetes Deployment file can say that an application
should have three replicas. Kubernetes and its controllers work to
maintain that state.

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-app
          image: nginx:1.27
```

In this example, `replicas: 3` describes the desired number of
application copies.

### 2. Versioned and Immutable History

Store the desired configuration in Git.

This gives you a history of changes:

-   Who made a change?
-   What was changed?
-   When was it changed?
-   Which commit introduced it?

Git history makes changes easier to review, trace, and audit. A commit
can also be reverted if a change causes a problem.

### 3. Automated

A GitOps controller regularly checks Git and the Kubernetes cluster.

Depending on its configuration, it can apply changes automatically or
wait for someone to start a sync.

Automation reduces repeated manual work and helps keep environments
consistent.

### 4. Observable

You should be able to see whether your application is working as
expected.

GitOps tools such as Argo CD show information about:

-   Application sync status
-   Application health
-   Differences between Git and the cluster
-   Deployment errors

Git history also helps you understand how the configuration changed over
time.

------------------------------------------------------------------------

## 1.4 GitOps vs Traditional CI/CD

Traditional CI/CD pipelines and GitOps can work together. They usually
handle different parts of the delivery process.

For example:

-   **CI (Continuous Integration)** builds and tests the application.
-   **CD with GitOps** makes sure Kubernetes runs the version described
    in Git.

  -----------------------------------------------------------------------
  Feature                 Traditional push-based  GitOps with Argo CD
                          deployment              
  ----------------------- ----------------------- -----------------------
  **How changes reach the A pipeline sends        Argo CD pulls
  cluster**               changes to Kubernetes.  configuration from Git
                                                  and applies it.

  **Source of truth**     Depends on the pipeline Git stores the desired
                          and deployment setup.   configuration.

  **Cluster access**      The pipeline usually    Argo CD runs in or
                          needs credentials to    connects to the cluster
                          access the cluster.     and uses its configured
                                                  permissions.

  **Rollback**            Often done by running a Revert the Git change
                          deployment again with   or restore an earlier
                          an older version.       configuration, then
                                                  sync it.

  **Drift detection**     May need extra tools or Argo CD continuously
                          manual checks.          compares desired and
                                                  live state.

  **Change history**      Often spread across     Git provides a
                          pipeline logs and other reviewable history of
                          systems.                configuration changes.
  -----------------------------------------------------------------------

### One Important Point

GitOps does not replace every CI/CD tool.

A common setup looks like this:

1.  A developer pushes application code to Git.
2.  A CI pipeline runs tests and builds a container image.
3.  The new image is published to a container registry.
4.  The deployment configuration in Git is updated with the new image
    version.
5.  Argo CD detects the configuration change.
6.  Argo CD syncs the change to Kubernetes, automatically or after
    approval, depending on the setup.

This keeps application building and deployment configuration management
as separate, connected steps.

------------------------------------------------------------------------

## 1.5 Why Use GitOps?

GitOps can help teams:

-   **Reduce manual work:** Fewer deployment steps need to be performed
    by hand.
-   **Track changes:** Git provides a clear history of configuration
    updates.
-   **Recover more easily:** Reverting a Git change can restore an
    earlier desired configuration.
-   **Find configuration drift:** Argo CD shows when the cluster differs
    from Git.
-   **Improve consistency:** Different environments can use reviewed and
    version-controlled configuration.
-   **Control changes:** Teams can use pull requests and code reviews
    before changes reach a cluster.

Remember that GitOps still needs good access control, secret management,
testing, and monitoring. It does not make a system secure or reliable
automatically.

------------------------------------------------------------------------

## 1.6 Quick Revision

  -----------------------------------------------------------------------
  Question                            Short answer
  ----------------------------------- -----------------------------------
  What is GitOps?                     Managing application and
                                      infrastructure configuration
                                      through Git.

  What is the source of truth in      The Git repository containing the
  GitOps?                             desired configuration.

  What is Argo CD?                    A GitOps continuous delivery tool
                                      for Kubernetes.

  What is desired state?              The configuration that describes
                                      how the system should run.

  What is live state?                 The resources currently running in
                                      the cluster.

  What is drift?                      A difference between desired state
                                      and live state.

  Does Argo CD always sync            No. Automatic sync must be
  automatically?                      configured; manual sync is also
                                      possible.

  Does GitOps replace CI?             No. CI can still build and test the
                                      application.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Key Takeaways

-   Git is the source of truth for the desired configuration.
-   Kubernetes runs the application.
-   Argo CD compares the configuration in Git with the live cluster.
-   Sync applies the desired configuration.
-   Self-heal can correct drift when enabled.
-   CI and GitOps CD can work together in one delivery process.

-------
