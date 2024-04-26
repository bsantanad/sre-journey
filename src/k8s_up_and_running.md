# 🐋 k8s up and running

# Table of Contents

<!-- toc -->

# Introduction

<aside> 
💡 K8s is an open source orchestrator for deploying containerized applications
</aside>

It has become the standard API for building cloud native applications. It
provides the software needed for build and deploy, reliable, scalable
distributed systems.

**Distributed systems:** Most services nowadays are delivered via the network,
they have different moving parts in different machines talking to each other
over the network via APIs. 

**Reliable:** Since we rely a lot on these distributed systems, they cannot
fail, even if some part of them crashes, they need to be kept up and running,
and be fault-tolerant

**Availability:** They must still be available during maintenance, updates,
software rollouts and so on.

**Scalable:** We live in a highly connected society, so these systems need to
be able to grow their capacity without radical redesign of the distributed
systems.

Most of the reasons people come to use containers and container orchestration
like k8s, can be boil down to 5 things:  **velocity, scaling, abstracting your
infra, efficiency, cloud native ecosystem**

## Development Velocity

Today software is delivered via de network more quickly than ever, but velocity
is not just a matter of raw speed, people are more interested in a highly
reliable service, all users expect constant uptime, even as the software is
being updated. 

Velocity is measures in term of the number of things you can ship while
maintaining a highly available service.

k8s give you the tools to to do this, the core concepts to enable this are:

- **Immutability**
    
    Software used to be mutable (imperative), you installed `vim` using `dnf`
    and then one day you would do `dnf update`  and it would add modifications
    on top of the binary that you already have for `vim`. This is what we call
    *mutable* software.
    
    On the other hand, *immutable* software, you would not add on top of the
    previous release, you would build the image from zero. Which is how
    containers work. Basically when you want to update something in a
    container, you do not go inside it and do `dnf update vim` you create a new
    image with the version of vim you want, destroy the old one, and create a
    container with the new one.
    
    This makes rollbacks way easier, and it keeps tracks of what changed from
    version to version.
    
    <aside> 
    💡 Imperatively change running containers (going into the container and
    update a package for example) is an anti-pattern, and should be avoided.
    </aside>
    
- **Declarative Configuration**
    
    This is an extension of immutability but for the configuration. It
    basically means that you have a file where you defined the ideal state of
    your system. The idea of storing this files in source control is often
    called “infrastructure as code”. 
    
    The combination of declarative state stored in a version control system and
    the ability of k8s to make it  match the state makes rollback super easy.
    
- **Self-Healing Systems**
    
    K8s will continuously take action to make sure the state on the declarative
    configuration is met. Say it has declared 3 replicas, then if you go and
    create one extra, it will kill it, and if you were to destroy one, it would
    create it. 
    
    This means that k8s will not only init your system but will guard it
    against any failures that might destabilize it. 
    

## Scaling Your Service and Your Teams

WIP

# Deploying a k8s cluster

You can use a cloud provider, deploying on  bare-metal quite hard, other good
alternatives
[http://kind.sigs.k8s.io](http://kind.sigs.k8s.io/docs/user/configuration/)
that has a really cool logo, and works on using containers for the nodes.

