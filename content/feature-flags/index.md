---
date: 2016-04-01T20:08:11+01:00
title: Feature flags
weight: 41
---

Feature Flags are a time-honored way to control the capabilities of an application or service in a large decisive way. 

### An Example

Say you have 
an application or service that launches from the command-line that has a `main` method or function. Your feature flag 
could be `--withOneClickPurchase` passed in as a command-line argument. That could activate lines of code in the app to 
do with Amazon's patented one-click purchasing 
experience.  Without that command-line argument, the application would run with a shopping cart component. At least
that's the way the developers coded that application. The 'One Click Purchase' and 'Shopping Cart' alternates are 
probably also the same language that the business people associated with the project use. It gets complicated in 
that flags need not be implicitly a/b or new/old, they could be additive. In our case here, there could also be a
`--allowUsersToUseShoppingCartInsteadOfOneClick` capability. Flags can be additive, you see.

{{< note title="Flags Are Toggles" >}}
Industry Luminary, Martin Fowler, calls this technique 'Feature Toggles', and wrote a foundational definition (see refs below). 
Feature Flags is in wider use by the industry, though, so we're going with that.
{{< /note >}}

## Granularity

It could be that the flag controls something large like the UI of a component. In our case above we could say that 
`OneClickPurchasing` and `ShoppingCart` are the names of components.  It could be that the granularity of the flag
is much smaller - Say Americans want to see temperatures in degrees Fahrenheit and other nationalities would 
prefer degrees Centigrade/Celsius. We could have a flag `--temp=F` and `--temp=C`. For fun, the developers also added
`--temp=K` (Kelvins).

## Implementation

For the `OneClickPurchasing` and `ShoppingCart` alternates, it could be that a `PurchasingCompleting` 
abstraction was created. Then at the most primordial boot place that's code controlled, the `--withOneClickPurchase` flag
is acted upon:

Java, by hand:

```java
if args.contains("--withOneClickPurchase") {
  purchasingCompleting = new OneClickPurchasing();
}
```

Java Dependency Injection via config:

```java
bootContainer.addComponent(classFromName(config.get("purchasingCompleting")));
```

There are many more ways of passing flag intentions (or any config) to a runtime.  If you at all can, you want to 
 avoid if/else conditions in the code where a path choice would be made. Hence our emphasis on an abstraction.

## Continuous Integration pipelines

It is important to have CI guard your reasonable expected permutations of flags. That means tests that happen on an
application or service after launching it, should also be adaptable and test what is meaningful for those flag 
permutations. It also means that in terms of CI pipelines there is a fan-out **after** unit tests, for each meaningful
flag permutation. A crude equivalent is to run the whole CI pipeline in parallel for each meaningful flag permutation.
That would mean that each commit in the trunk kicks off more than one build - hopefully from elastic 
infrastructure.

## Runtime switchable

Sometimes flags set at app launch time is not enough. Say you are an Airline, selling tickets for flights online.
You might also rent out cars in conjunction with a partner - say 'Really Cool Rental Cars' (RCRC). The connection to 
any partner or their up/down status is outside your control, so you might want a switch in the software that works 
without relaunch, to turn "RCRC partner bookings" on or off, and allow the 24&#47;7 support team to flip it if certain 'Runbook' conditions
have been met.  In this case, the end users may not notice if Hertz, Avis, Enterprise, etc are all still amongst
the offerings for that airport at the flight arrival time.

Key for Runtime switchable flags is the need for the state to persist. A restart of the application or service should
not set that flag choice back to default - it should retain the previous choice. It gets complicated when you think
about the need for the flag to permeate multiple nodes in a cluster of horizontally scaled sibling processes. For
that last, then holding the flag state in Consul{{< ext url="https://www.consul.io" >}}, 
Etcd{{< ext url="https://github.com/coreos/etcd" >}} (or equivalent) is the modern way.

## Build Flags

Build flags affect the application or service as it is being built. With respect to the `--withOneClickPurchase` flag again,
the application would be incapable at runtime of having that capability if the build were not invoked with the suitable
flag somehow.

## A/B testing and betas

Pushing code that's turned off into production, allows you to turn it on for ephemeral reasons - you want a subset of 
users to knowingly or unknowingly try it out. A/B testing (driven by marketing) is possible with runtime flags. So is 
having beta versions of functionality/features available to groups.

## Tech Debt - pitfall

Flags get put into codebases over time and often get forgotten as development teams pivot towards new business deliverables.
Of course, you want to wait a while until it is certain that you are fixed on a toggle state, and that's where the 
problem lies - the application works just fine with the toggle left in place, and the business only really cares
about new priorities. The only saving grace is the fact that you had unit tests for everything, even for code that
is effectively turned off in production. Try to get the business to allow the remediation of flags (and the code
they apply to) a month after the release. Maybe add them to the project's readme with a "review for delete" date.

## History

Some historical predecessors of feature toggles/flags as we know it today:  

- Unified Versioning through Feature Logic (Andreas Zeller and Gregor Snelting, 1996){{< ext url="http://www.cs.tufts.edu/~nr/cs257/archive/andreas-zeller/tr-96-01.pdf" >}} - white paper.
- Configuration Management with Version Sets: A Unified Software Versioning Model and its Applications (Andreas Zeller's, 1997){{< ext url="https://www.st.cs.uni-saarland.de/publications/files/zeller-thesis-1997.pdf" >}} - Ph.D. thesis.

There's a warning too: 

- "#ifdef considered harmful" (Henry Spencer and Geoff Collyer, 1992){{< ext url="http://www.literateprogramming.com/ifdefs.pdf" >}} - white paper.

Brad Appleton says:

<br><div style="padding-left: 45px; padding-right: 45px"><span style="font-size: 150%">&ldquo;</span>
The thing I do not like about feature-toggles/flags is when they end up NOT being short-lived as intended, 
and we end up having to revisit Spencer and Collyer's famous paper. The funny thing is feature-branches 
started out the same way. When they were first introduced it was for feature-teams using very large features, and the 
purpose of the separate branches was because too many people were trying to commit at the same time to the same branch. 
So the idea was use separate branches (for scale) and teams would integrate to their team-branch daily or more often 
WITH at least nightly integration across all feature-branches [sigh].
</div>

# References elsewhere

<a id="showHideRefs" href="javascript:toggleRefs();">show references</a>

<div>
    <table style="border: 0; box-shadow: none">
        <tr>
            <td style="padding: 2px" valign="top">29 Oct 2010, MartinFowler.com article</td>
        </tr>
        <tr>
            <td style="border-top: 0px; padding: 2px" valign="top"><a href="https://martinfowler.com/bliki/FeatureToggle.html">Feature Toggle</a></td>
        </tr>
    </table>
    <table style="border: 0; box-shadow: none">
        <tr>
            <td style="padding: 2px" valign="top">30 May 2011, TechCrunch article</td>
        </tr>
        <tr>
            <td style="border-top: 0px; padding: 2px" valign="top"><a href="http://techcrunch.com/2011/05/30/facebook-source-code">The Next 6 Months Worth Of Features Are In Facebook's Code Right Now, But We Can't See</a></td>
        </tr>
    </table>
    <table style="border: 0; box-shadow: none">
        <tr>
            <td style="padding: 2px" valign="top">19 Jun 2013, Slides from a talk</td>
        </tr>
        <tr>
            <td style="border-top: 0px; padding: 2px" valign="top"><a href="http://www.slideshare.net/cb372/branching-strategies">Branching Strategies: Feature Branches vs Branch by Abstraction</a></td>
        </tr>
    </table>
    <table style="border: 0; box-shadow: none">
        <tr>
            <td style="padding: 2px" valign="top">10 Oct 2014, Conference Talk</td>
        </tr>
        <tr>
            <td style="border-top: 0px; padding: 2px" valign="top"><a href="https://www.perforce.com/merge/2014-sessions/trunk-based-development-enterprise-its-relevance-economics">Trunk-Based Development in the Enterprise - Its Relevance and Economics</a></td>
        </tr>
    </table>
    <table style="border: 0; box-shadow: none">
        <tr>
            <td style="padding: 2px" valign="top">08 Feb 2016, MartinFowler.com article</td>
        </tr>
        <tr>
            <td style="border-top: 0px; padding: 2px" valign="top"><a href="https://martinfowler.com/articles/feature-toggles.html">Feature Toggles</a></td>
        </tr>
    </table>
    <table style="border: 0; box-shadow: none">
        <tr>
            <td style="padding: 2px" valign="top">23 May 2017, DevOps.com article</td>
        </tr>
        <tr>
            <td style="border-top: 0px; padding: 2px" valign="top"><a href="https://devops.com/feature-branching-vs-feature-flags-whats-right-tool-job/">Feature Branching vs. Feature Flags: What’s the Right Tool for the Job?</a></td>
        </tr>
    </table>
    
</div>
