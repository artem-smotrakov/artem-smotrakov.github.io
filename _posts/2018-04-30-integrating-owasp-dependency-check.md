---
layout: post
title: Integrating OWASP Dependency Check in to development process
date: 2018-04-30 22:14:01.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Java
- Security
- Web security
permalink: "/en/security/integrating-owasp-dependency-check.html"
---
OWASP Dependency Check is a well known open-source tool which can track dependencies in your project and identify components with known published vulnerabilities. The tool supports multiple languages and platforms such as Java, .NET, Ruby and Python. One of the simplest ways how you can use Dependency Check in your project is just to run it manually. This way has at least one disadvantage: you have to make sure that you run the tool regularly. Fortunately there is a couple of ways how you can automate this process.

But unfortunately sometimes it's not enough just to automate something. If the tool reports a vulnerability it means someone has to fix it. At least it would be good to evaluate the problem. In a perfect world, all issues are addressed immediately, but in the real world, development teams always have no time for that. Besides integrating Dependency Check to CI/CD, there may be a couple of other steps to get vulnerable dependencies updated.

## Integrating OWASP Dependency Check in to CI/CD

First, you can integrate the tool in to your favorite build system. There are plugins for Gradle, Maven and others which allow you to easily include Dependency Check in to your build and test routines. Next, there is a plugin for Jenkins which allows you to integrate the tool to you CI/CD pipelines. As a result, you can get Dependency Check reports in your Jenkins. If your Jenkins is configured to run a job for each commit, branch or pull request, then most likely you're going to get a notification about vulnerable components once the information about them becomes public. There is one exception. If you don't update your application often, but it's still deployed, then there is a chance that you don't get a notification in time. To solve this, you can configure Jenkins to run Dependency Check regularly (for example, every night).

## Problems with updating vulnerable dependencies

As I mentioned before, integrating OWASP Dependency Check with Jenkins may not be enough to keep dependencies up-to-date.

First, we need to make sure that someone reads the reports and evaluates identified vulnerabilities. But an engineer can forget to take a look at the report, or maybe even ignore it assuming that someone else is going to do that. Then, someone else can also forget about it, or think the same. As a result, after some time we can figure out that there have been vulnerable components for X months even though we integrated Dependency Check with Jenkins.

Second, we need to make sure that someone actually updates vulnerable components after they were found. In a perfect world, it can be done right away, but in reality we can have a couple of problems.

Let's assume that our development team is working on important feature X which needs to be delivered soon. The deadline is coming. Just in time Dependency Check identifies a vulnerability. But updating the vulnerable library has a risk of introducing a regression, compatibility issue, etc. One way to mitigate this risk is to have good testing procedures. Ideally, it should an automated test suite which runs in your CI/CD.&nbsp; If a full test cycle requires significant time, then the delivery of feature X may be delayed which doesn't make us happy. Or, maybe we just don't have good test coverage.

Next, updating one library may cause updating others. This is not too bad in general, but it may increase a bit the risk of introducing a regression. Plus, new versions of libraries may change API which can result to updating our code. Again, it requires time, increases the risk of regressions and so on. in the end it can delay the delivery of our important feature X.

Furthermore, vulnerabilities in components can have different severity levels and impact. Some of them may even have no impact for a project. Plus, we should not forget that OWASP Dependency Check can report a false-positive. It all means that a reported issue should be evaluated at first, and it may turn out that the issue is not that critical and it's not necessary to delay the feature X trying to fix this issue.

## Setting up a process of updating vulnerable dependencies

On the one hand, in the real world sometimes it's not a good idea to enforce updating vulnerable components once they were identified. On the other hand, we need to make sure that we don't forget to updated them a bit later.

Setting up a process of updating vulnerable components may include the following steps:

1. Integrate OWASP Dependency Check to build system and Jenkins:
  1. The check should be included to a regular job which is triggered by a commit or pull request.
  2. The check should be run regularly in a separate job.
2. Introduce a quality gate which fails jobs if a vulnerability with high severity level was found. Let's refer to [NIST](https://nvd.nist.gov/vuln-metrics/cvss), and say that a vulnerability has a high severity level if its CVSS score is higher than 7.
3. Once a job failed, an engineer should evaluate reported vulnerabilities. A security engineer may be involved here as well. If they decide that the vulnerabilities are dangerous enough, then the vulnerable components should be updated immediately.
4. If they decide that the vulnerabilities are not that critical for the project, then they do the following:
  1. File a ticket for updating vulnerable components.
  2. Add a record for the identified vulnerabilities to a [suppression file](https://jeremylong.github.io/DependencyCheck/general/suppression.html) to stop the build failing. The record should include a link to the ticket. Here we use the suppression file as a list of confirmed known failures which need to be fixed.
5. Review a list of tickets regularly, and plan fixing vulnerable components.

This process is trying to achieve a balance between security and development. On the one hand, there is a way how we can postpone updating vulnerable dependencies if it's acceptable. On the other hand, we have a clear backlog for updating vulnerable components, so we can control the situation a bit better. Of course, this process doesn't magically solve the problem - in the end someone needs to go and update the libs.

## Implementing a quality gate with OWASP Dependency Check

How to set up the quality gate defined above? Fortunately it's pretty simple. OWASP Dependency Check has an [option](https://jeremylong.github.io/DependencyCheck/dependency-check-cli/arguments.html) that sets a CVSS score and instructs the tool to return a non-zero exit code if a vulnerability with a CVSS score equal to or higher was identified. Plugins for Gradle, Maven and Jenkins supports this option. Here is an example of `build.gradle` file which introduces such a quality gate:

<script src="https://gist.github.com/artem-smotrakov/c68930234da6b6f308d3cb2534472e3b.js"></script>

To make a Dependency Check report available in Jenkins, you need to run `dependencyCheckPublisher`&nbsp;task in your Jenkins pipeline. Once the quality gate is configured, a job is going to fail if a vulnerability was found which means that the steps after Dependency Check may not be run. You need to make sure that `dependencyCheckPublisher`&nbsp;is always run even if the quality gate failed. Here is an example of Jenkinsfile:

<script src="https://gist.github.com/artem-smotrakov/31f5c1911041268797cc6d7a90be5e93.js"></script>

There is another nice improvement which can make our life a bit easier. OWASP Dependency Check uses the [National Vulnerability Database](https://nvd.nist.gov/) (NVD). The tool downloads the whole database when it runs for the first time. Then, the tool tries to get updates regularly. Unfortunately it turns out that the NVD may be down sometimes. Dependency Check is going to fail when the NVD is not available. If we have the quality gate defined above, then jobs are going to fail as well which may block our CI/CD process. But that's not what we want. To address this problem, most of plugins provide an option to ignore this kind of issues. But it means that we're going to skip the check completely if the NVD is down. Fortunately there is a better solution. You can configure a separate Jenkins job which regularly downloads a copy of the NVD (for example, it can run every night). Then, you can [configure other jobs to use the copy](https://jeremylong.github.io/DependencyCheck/data/mirrornvd.html).

In conclusion, here you can find [an example of Spring Boot application with the quality gate described above](https://github.com/artem-smotrakov/spring-boot-fun/releases/tag/v1.0). Enjoy!

