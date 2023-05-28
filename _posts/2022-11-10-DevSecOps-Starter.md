---
layout: post
title: "Starting in DevSecOps"
author: Isaac GC
tags: [ DevSecOps, DevOps, Security]
excerpt: |
  A personal accounting and overview of DevSecOps
---

---

### Quick Overview

Suppose your boss says that you need to implement or improve the security within DevOps environments. It is highly likely that your company doesn't have a Security Engineer or someone with prior experience in DevSecOps, or maybe its something you want to break into as a future career path. As the DevOps role can generally be referred to as a new "SysAdmin", adding in Security responsibilites can seem daunting.

This can be overwhelming with not knowing where to start, or maybe you just need a reference that helps identify areas to focus on. This will be the first part of a series of blog posts that hopefully will give a starting point to build of off.

As a quick disclaimer, *these posts will be based off my personal views/observations and in no way reflect upon previous employers or companies I have worked with. These posts are meant to take the learning experiences I have obtained and share them out for those that come across the same situtations. While examples may be given, these should not be relied on as complete solutions or the right solution for your situation.*

---

### Getting Started

As a "hello world" place to get started in DevSecOps or even just CyberSecurity, you probably have an idea of the operations needed to deploy systems or applications. Ironically enough, the acronym "SDLC" fits both Systems Development Life Cycle and Software Development Life Cycle. Generally when talking about DevSecOps, Software is the main focus, but in major businesses and the government, this can also refer to the systems/servers deployed internally. However, irregardless of if SDLC refers to Software or Systems, the same concepts can be adapted to address both.

#### <u>The Supply Chain (Dependencies)<u>

As a tangible place to get started with addressing security, the first area to look at will always be the supply chain. The supply chain itself is one of the hardest pieces to address and most frustrating.

In software, this usually is is related to dependencies, built by other developers, used in other applications. Sometimes the application is affected by "dependency hell", where there are several dependencies that share the same library, but different versions of that library, resulting in multiple duplicates of the library. Depending on how many dependencies are used, the application could blow up to be a bloated massive thing where a single dependency update could break the application through dependency conflicts or even circular dependencies.

If that weren't bad enough, malicious dependencies are pushed daily or even known-good dependencies are updated with malicious code. Some recent well-known examples of this is the 2020 Solarwinds attack or even the 2021 Log4J widespread vulnerability. I won't go into these as many others have done great write-ups of these attacks and I highly recommend you search for them when you get chance if you haven't already. The question resulting from these or similar attacks is, how do you defend against them?

The answer is that there really isn't a way. The current method used to combat it is to use a dependency scanning tool, such as Snyk, OWASP Dependency-Check, or one of the many other tools available.

These tools constantly follow the latest released lists of vulnerable dependencies which guard you against *known* vulnerabilities, but what about *unknown* vulnerabilities? At this point, it is entirely possible that you may be saying "Duh, of course you can't protect against unknown vulnerabilities, that's because they're *unknown*." And I agree with this, at least in part, as you can't know what you can't know. Thankfully, there are a lot of security researchers continuously looking for and finding these dependencies.

This includes everything from software itself to even the sourcing of hardware (where additional malicious chips may be installed -> [See this article for more info on that](https://arstechnica.com/information-technology/2019/10/planting-tiny-spy-chips-in-hardware-can-cost-as-little-as-200/)). Yet, for you to defend against it, having a Bill-of-Materials (or BOM) is extremely handy, and highly recommended if nothing more than to track the top-level dependencies used in the application(s) you support. You should also consider using another tool like jFrog Artifactory or Sonatype's Nexus Repository to proxy the dependencies/software-artifacts and keep track of them that way as well. (A more in-depth article on a BOM and Sonatype Nexus Repository will be created later). Also by using tools like Sonatype Nexus Repository, your build's can be made that much quicker as the dependencies would be located much closer to the deployment pipelines. :)

#### <u>Static Application Security Testing (SAST)</u>

There are many tools out there that perform SAST testing, but in my experience, none that are commercially available do it right and *if* they do it right, they usually are locked behind having to go through the company's sales people (which is a nuisance) and even then sometimes don't live up to the hype. Yes, there are some tools that are commonly used/known such as SonarQube, but I have found they often fall short of the Security aspect of Static Code Analysis (SCA). The best ones I have seen so far is actually SemGrep or Snyk, yet with Snyk especially, it becomes prohibitively expensive to afford, especially for smaller and/or young companies.

It is once you dive into Code Review that you should immediately be able to tell why SAST testing is so hard to set up or get right. Code Review is difficult enough on its own, let alone the many different ways it could be written to address a certain issue/function. Being able to read these many styles of code, understand what is happening, then writing rules that address all the writing styles is a major challenge on its own. SonarQube does this mostly well, with SemGrep following closely behind, yet both still not completely on par with the level most need.

I won't go into SAST testing more than this as I plan on making a longer blog post about the in's-and-out's of SAST manual testing as well as automated testing. Yet, I will recommend if you are just getting started, to use SonarQube initially, then migrate to using SemGrep when you have more experience with what needs to be tested and how.

#### <u>Dynamic Application Security Testing (DAST)</u>

This is probably the most important part of the whole SDLC lifecycle as well as doing DevSecOps tasks. Whereas SAST focuses on the raw code (i.e. Static side of the house), DAST is instead performed on the running application. Irregardles about how the code is compiled, bundled, and/or ran, the running application *will*, at some level, be transformed from the language it was written to machine code where the computer can process it. While in some cases it may be possible to view vulnerable code via SAST methods, DAST methods can expose vulnerabilities that live in your application that are instead caused by referenced dependencies.

Testing with this methodology would be considered "black-box" testing as the DAST tool has no idea what the underlying code is doing, and instead throws random "junk" at the running application to see how it reacts. Depending on what specifically is being tested, this usually is referred to as fuzzing (sending malformed/incomplete data to find bugs/issues), and *can but not always* consist of also performing load tests. Burp Suite is one example of the these tools that specifically targets web applications and allows automated testing for many known vulnerabilites, as well as most importantly, the OWASP 10.

#### <u>CI/CD Pipelines</u>

The CI/CD pipelines are one of the most critical points in the applications lifecycle. If it were to happen, this is where integrity of the entire CI/CD pipeline can be compromised. As this is where the application is built with its dependencies, it is the prime point for dependencies served by malicious actors to be introduced to or within the application. These can be dependencies of the application, the dependencies of those dependencies, or even the tools used to build the application, such as Jenkins and/or Docker.

While the security of the dependencies of the application can be addressed through Supply Chain tools/security, the other two are a bit more difficult to defend against. In the case of a dependency's dependency, an example being the `psycopg2` in Python, both the PostgreSQL source code and the psycopg2 source code need to be downloaded, then compiled (especially if one wants to enable SSL). The source code for both also have extra dependencies that need to be downloaded from the system's package manager such as `libpq`. If the sources for these are not known and tracked, having a Bill-of-Materials (BOM) would next to useless, even if you kept all the application's dependencies up to date.

This is further exacerbated when you consider Docker and automation servers like Jenkins. There are tons of Docker containers out on the internet between, most notably between the Quay.io and DockerHub repositories. Any container you don't control should be considered dangerous, especially containers meant for base images. Thankfully in DockerHub's case, there is a badge next to the image's name that says `Docker Official Image` or `Verified Publisher` and these *should* be considered safe. (I'd still recommend investigating them if not for anything else other than learning their internals and how they were built) There is also a third badge, `Sponsored OSS` (anybody can apply for/get it without their application/image being truly verified), which should **absolutely never be trusted** and should instead be regarded with the same lack of trust as any other Docker image listed without badges.

In pipeline automation tools such as Jenkins, Docker images, custom scripts, even custom in-house libraries are used in the process to build the applications. To help protect within the CI/CD pipelines, *everything* should be able to be audited or made to be auditable. Everything should also be configured to have or be a disposible environment and with least-privilege as much as possible, *including* any Docker build images used. (Don't use root in the containers, make a new less-privileged user!)

The easiest way to do this is to simply save to a version control system such as Git or SVN everything that can feasibly be saved. In these cases, it is best to use your judgement. For example, you should save third-party scripts (to prevent arbitrary changes) locally in Git, but you shouldn't save packages from the APT Package Manager. As most in-house scripts will be saved to the Git repository, that also means the reference to the Docker image to build the application or the image the application runs in should/could also be saved to the Git repository.

An easy way to integrate security in the described mess above is to split apart the stages into various sections. In the initial stage, you could check the dependencies of the application and if there are violations of your company's or team's policies, you would fail the build. In a subsequent stage, if you pull down a Docker image to build the application, you would scan the image with a tool such a Trivy, and if that too fails your company's/team's policies, you would fail that build too. This stage could also be repeated for the final image used to run the application (if the application runs in a container). Finally, when the build completes, a final build job should be queued to run DAST scans against the final image or application. If the DAST scans do not pass accordingly, builds should not be allowed to built for that version of the application in production environments, and a report of the findings should be made available so that the developers can resolve the issues.

#### <u>Final Stages/Running Application</u>

In the running application environment, especially if the environment is open to the internet (which it *should really not be* for non-production environments), you should consider building additional security around the application. Simple things such as implementing network rules to only allow the required applications to talk amongst each other, not sharing networks/security-rules between non-production/production environments, and removing all administrator permissions except for break-glass emergencies in production environments. Half the battle of defending the environment is just denying access, preventing further or inadvertant access from occuring. The other half of the battle, which may or may not be simple depending on your company/team, is simply to ***LOG EVERYTHING***.

It doesn't matter whether you actually store the logs long-term, it is more of a matter of being *able* to see everything. In fact, once you accomplish the challenge of logging everything, the next thing you'll have is the challenge to filter everything (this will be a future blog post too). It doesn't matter whether you are providing a web service through a cloud provider, or if your application is local to the machine, *everything* should be logged (make sure sensitive data is sanitized) as well, if nothing else, for troubleshooting/investigations. 

To further add layers to the security onion of your environment, it is important to invest in other solutions like a Web Application Firewall (WAF) and some sort of DDoS solution (CloudFlare is pretty common for this part). The WAF will help prevent both automated (i.e. bots) and manual exploitation (human attackers) by blocking malicious requests. In the case like the Log4J nightmare in Dec 2021, a WAF rule could easily be added to prevent the malicious strings from being injected, while buying time for the developers to fix the affected code. You should also consider adding endpoint security to any server (non-containerized) running a full OS such as Debian, Ubuntu, Redhat, or SUSE.

---

### Summary / Other Notes

Hopefully, if you've read this far, you would notice that I use terms such as "should" and "could". The reason behind is that simply stating or giving ideas in a blog post removes the learning curve behind understanding the topic further. These posts should be the spark that gets you to ask "how" or "why", and then having you dig deeper.

Between all of my previous job positions, I have found that simply asking the "how" or "why" is what makes a good engineer, irregardless if you are doing security, programming, or even simply on help desk support. Having that attitude with the ability to admit you are wrong and actually listening to other peoples sides/opinions is what makes the best engineers, not to mention really good people to work with. These are skills/traits are usually hard-won and take work to implement/learn, don't expect to have it all overnight, just take it day-by-day until you do. As my grandfather said when I was little "The only time you stop learning and growing is the day you die, so learn everything you can."

With the above mentality, you can go far. Like, *really* far. Keep learning, even if you make mistakes, just don't stop asking questions. Burn out in cybersecurity is real, so take care of yourself like reading a book, walking in nature, just do something not related to work. If your company treats you poorly, move on as they will almost always be able to replace you as much as you can replace them with a better company.

<u>Above all else, know this:</u>

Regardless of the company you work for, that company <u>will</u> be breached. In fact, it is a common saying in Cybersecurity that there are two types of companies:
> One that has been breached and the other that just doesn't know it yet

The goal you should strive for is the *how much* you are impacted and minimize that as much as possible. If all else, stay vigilant by watching for vulnerabilities, know as much as you can about whats happening in your environment, and keep learning to try to identify improvements.

---

#### Tools

These are some of the tools that I have used in my career. (Note: This is based off my experience and I am in no way affiliated with these companies)

- Supply Chain
  - Dependabot (limited to Github)
  - Github Actions (used with Dependabot and OWASP Dependency-Check for automated workflows)
  - Nexus Repository
  - Nexus Firewall
  - OWASP Dependency-Check
  - Snyk
  - Trivy

- SAST
  - SemGrep
  - Snyk
  - SonarQube

- DAST
  - Burp Suite (both Desktop and Server/Enterprise editions)
  - Nikto
  - Nessus / Tenable.io (technically a vulnerability scanner, but does scan the network for vulnerabilities)
  - OpenVAS (if you can, use nessus/tenable.io instead)
  - OWASP Zap

- Production / Cloud-Provider Environments
  - AWS WAF
  - Cisco Firewall
  - CloudFlare DDoS protection
  - ELK Stack (Elasticsearch/Kibana/Elastic-Agent for endpoint security, monitoring, and log-aggregation)
  - GrayLog (Log-aggregation)
  - Nessus / Tenable.io (Vulnerability Scanner)
  - Palo Alto (Firewall and WAF)
  - Splunk (Log-aggregation)
