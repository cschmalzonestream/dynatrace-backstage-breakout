# dynatrace-backstage-breakout
Get started with the Dynatrace Backstage plugin to enhance the developer experience. See how we used it as a one-stop shop for developers to see their applications and services and proactively gather performance information, such as error logs and quality gate validations.

**Table of contents:**

- [Overview](#overview)

## Overview

The usage of the Dynatrace Backstage plugin spawned from wanting to alert on performance degradation from a Site Reliability Guardian (SRG). I wanted the proof of concept (PoC) to be easy to grasp so I decided to containerize an ASP.NET Core weather application. 

This application queried the weather for a given city and would display the results to the user. To showcase the SRG, two versions of this application were created, main & feature, with feature introducing latency to the API call to the weather service. The SRG would detect this given its set SLOs and alert the developer of the performance degradation they introduced to the source code.

I then thought to myself, “How cool would it be if the developer was able to view these quality gate validations alongside performing their daily tasks? This was the birth of selecting an Internal Developer Platform (IDP) and the Dynatrace Backstage plugin.
