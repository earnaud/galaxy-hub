---
title: Public Galaxy Servers BoF
---
<slot name="/events/gcc2013/header" />



import links from "../../links.json"
<link-box :links="links" />
<slot name="/events/gcc2013/bof/linkbox" />

<div class='left'><a href='/events/gcc2013/bof/'><img src="/images/logos/GCC2013BoFLogo.png" alt="" width="160" /></a></div>

This page describes the **Public Galaxy Servers** [Birds of a Feather](/events/gcc2013/bof/) meetup being held at [GCC2013](/events/gcc2013/).

The BoF is for people who maintain, manage, or are otherwise responsible for [publicly accessible Galaxy servers](/use/).  By *publicly accessible*, we mean that anyone regardless of location or affiliation can do work on that Galaxy server.  (Servers can have quotas, require an account to be created, etc.)

We'll cover topics that are relevant to public sites, including

* Dealing with abuse
* Quotas
* Security
* Support
* ...

## When and Where

The [tentative time and location](/events/gcc2013/bof/#bof-schedule) for this BoF is Sunday evening, [at the Escape Pub](/events/gcc2013/program/#escape-to-the-pub).  The [Training Day](/events/gcc2013/training-day/) ends at 17:00; lets start around 5:30pm.


## Who is Participating

If you are interested, please add your name below and/or send an email to [Dave Clements](mailto:clements AT galaxyproject DOT org).

* [Dave Clements](/people/dave-clements/), Galaxy Project
* [Dorine Francheteau](/galaxy-team/), Galaxy Project
* Eric Kuyt, Central Veterinary Institute of Wageningen UR (CVI)
* Jacqueline Lee, University of Calgary
* Ron Horst, University of Queensland, Australia
* Gianmauro Cuccuru, CRS4
* Christos Kannas, University of Cyprus
* Ravi Madduri, U Chicago
* YiLei Wu, biodata limited
* Simon Gladman
* Clare Sloggett

## Notes

The nominal organizer of the BoF wanted this to be a discussion of issues that are specific to [publicly accessible Galaxy](/use/) servers (an admittedly hard thing to define - servers that are accessible to everyone at a large university aren't public *per se*, but they face almost all of the same issues.)

However the group had other plans and the session became about "Feature requests for things that really need to be addressed."

Those included:

* Security and Billing
  * Galaxy is not designed with security in mind from the ground up.
  * Authentication needs to be more pluggable.
  * Galaxy lacks the reporting feature on cpu hours which is sometimes very useful for funding agency. 
  * "We're just in accounting hell at the moment"
* Releases  
  * Every two months is too often for releases
    * Galaxy lacks a stable release scheme (~twice per year) which makes the life of public Galaxy admin a lot easier.
  * Better Versioning
* Adding an optional phone home feature to install script is going for sustainability of the project.  Being able to say who uses it is a good way to keep being funded.
* Tool shed is *currently* making things more complicated.
* Dataset profiligation.  Can easily end up with 3 copies of most of your datasets, just to get files into Galaxy.
* maintaining a public Galaxy server well requires minimal 0.5 fte. 

## Questions?

Send them to [Dave Clements](mailto:clements AT galaxyproject DOT org).
