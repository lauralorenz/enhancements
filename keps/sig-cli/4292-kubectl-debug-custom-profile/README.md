<!--
**Note:** When your KEP is complete, all of these comment blocks should be removed.

To get started with this template:

- [ ] **Pick a hosting SIG.**
  Make sure that the problem space is something the SIG is interested in taking
  up. KEPs should not be checked in without a sponsoring SIG.
- [ ] **Create an issue in kubernetes/enhancements**
  When filing an enhancement tracking issue, please make sure to complete all
  fields in that template. One of the fields asks for a link to the KEP. You
  can leave that blank until this KEP is filed, and then go back to the
  enhancement and add the link.
- [ ] **Make a copy of this template directory.**
  Copy this template into the owning SIG's directory and name it
  `NNNN-short-descriptive-title`, where `NNNN` is the issue number (with no
  leading-zero padding) assigned to your enhancement above.
- [ ] **Fill out as much of the kep.yaml file as you can.**
  At minimum, you should fill in the "Title", "Authors", "Owning-sig",
  "Status", and date-related fields.
- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary" and "Motivation" sections.
  These should be easy if you've preflighted the idea of the KEP with the
  appropriate SIG(s).
- [ ] **Create a PR for this KEP.**
  Assign it to people in the SIG who are sponsoring this process.
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the KEP clarified and merged quickly. The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.

Just because a KEP is merged does not mean it is complete or approved. Any KEP
marked as `provisional` is a working document and subject to change. You can
denote sections that are under active debate as follows:

```
<<[UNRESOLVED optional short context or usernames ]>>
Stuff that is being argued.
<<[/UNRESOLVED]>>
```

When editing KEPS, aim for tightly-scoped, single-topic PRs to keep discussions
focused. If you disagree with what is already in a document, open a new PR
with suggested changes.

One KEP corresponds to one "feature" or "enhancement" for its whole lifecycle.
You do not need a new KEP to move from beta to GA, for example. If
new details emerge that belong in the KEP, edit the KEP. Once a feature has become
"implemented", major changes should get new KEPs.

The canonical place for the latest set of instructions (and the likely source
of this file) is [here](/keps/NNNN-kep-template/README.md).

**Note:** Any PRs to move a KEP to `implementable`, or significant changes once
it is marked `implementable`, must be approved by each of the KEP approvers.
If none of those approvers are still appropriate, then changes to that list
should be approved by the remaining approvers and/or the owning SIG (or
SIG Architecture for cross-cutting KEPs).
-->
# KEP-4292: Custom profiling support in kubectl debug command

<!--
This is the title of your KEP. Keep it short, simple, and descriptive. A good
title can help communicate what the KEP is and should be considered as part of
any review.
-->

<!--
A table of contents is helpful for quickly jumping to sections of a KEP and for
highlighting any additional information provided beyond the standard KEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Release Signoff Checklist](#release-signoff-checklist)
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposal](#proposal)
  - [User Stories (Optional)](#user-stories-optional)
    - [Story 1](#story-1)
    - [Story 2](#story-2)
    - [Story 3](#story-3)
  - [Notes/Constraints/Caveats (Optional)](#notesconstraintscaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
- [Design Details](#design-details)
  - [Test Plan](#test-plan)
      - [Prerequisite testing updates](#prerequisite-testing-updates)
      - [Unit tests](#unit-tests)
      - [Integration tests](#integration-tests)
      - [e2e tests](#e2e-tests)
  - [Graduation Criteria](#graduation-criteria)
    - [Alpha](#alpha)
    - [Beta](#beta)
    - [GA](#ga)
  - [Upgrade / Downgrade Strategy](#upgrade--downgrade-strategy)
  - [Version Skew Strategy](#version-skew-strategy)
- [Production Readiness Review Questionnaire](#production-readiness-review-questionnaire)
  - [Feature Enablement and Rollback](#feature-enablement-and-rollback)
  - [Rollout, Upgrade and Rollback Planning](#rollout-upgrade-and-rollback-planning)
  - [Monitoring Requirements](#monitoring-requirements)
  - [Dependencies](#dependencies)
  - [Scalability](#scalability)
  - [Troubleshooting](#troubleshooting)
- [Implementation History](#implementation-history)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
  - [Flags for all fields](#flags-for-all-fields)
  - [Customizable Pod Spec instead corev1.Container](#customizable-pod-spec-instead-corev1container)
<!-- /toc -->

## Release Signoff Checklist

<!--
**ACTION REQUIRED:** In order to merge code into a release, there must be an
issue in [kubernetes/enhancements] referencing this KEP and targeting a release
milestone **before the [Enhancement Freeze](https://git.k8s.io/sig-release/releases)
of the targeted release**.

For enhancements that make changes to code or processes/procedures in core
Kubernetes—i.e., [kubernetes/kubernetes], we require the following Release
Signoff checklist to be completed.

Check these off as they are completed for the Release Team to track. These
checklist items _must_ be updated for the enhancement to be released.
-->

Items marked with (R) are required *prior to targeting to a milestone / release*.

- [X] (R) Enhancement issue in release milestone, which links to KEP dir in [kubernetes/enhancements] (not the initial KEP PR)
- [x] (R) KEP approvers have approved the KEP status as `implementable`
- [X] (R) Design details are appropriately documented
- [x] (R) Test plan is in place, giving consideration to SIG Architecture and SIG Testing input (including test refactors)
  - [ ] e2e Tests for all Beta API Operations (endpoints)
  - [x] (R) Ensure GA e2e tests meet requirements for [Conformance Tests](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/conformance-tests.md) 
  - [x] (R) Minimum Two Week Window for GA e2e tests to prove flake free
- [X] (R) Graduation criteria is in place
  - [x] (R) [all GA Endpoints](https://github.com/kubernetes/community/pull/1806) must be hit by [Conformance Tests](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/conformance-tests.md) 
- [x] (R) Production readiness review completed
- [x] (R) Production readiness review approved
- [X] "Implementation History" section is up-to-date for milestone
- [X] User-facing documentation has been created in [kubernetes/website], for publication to [kubernetes.io]
- [X] Supporting documentation—e.g., additional design documents, links to mailing list discussions/SIG meetings, relevant PRs/issues, release notes

<!--
**Note:** This checklist is iterative and should be reviewed and updated every time this enhancement is being considered for a milestone.
-->

[kubernetes.io]: https://kubernetes.io/
[kubernetes/enhancements]: https://git.k8s.io/enhancements
[kubernetes/kubernetes]: https://git.k8s.io/kubernetes
[kubernetes/website]: https://git.k8s.io/website

## Summary

This proposal adds a new custom profiling feature on top of predefined profiles in `kubectl debug` command.

## Motivation

`kubectl debug` command provides a set of predefined profiles and users can pick the appropriate ones
according to their needs and roles. However, in some cases (maybe even in most cases), users might want 
to customize these profiles by adding:
* environment variables https://github.com/kubernetes/kubectl/issues/1486
* replicate volume mounts https://github.com/kubernetes/kubectl/issues/1071
* security contexts https://github.com/kubernetes/kubernetes/pull/113009
* labels https://github.com/kubernetes/kubectl/issues/1364, https://github.com/kubernetes/kubernetes/issues/115679
* image pull secrets https://github.com/kubernetes/kubectl/issues/1506

and many others. Users can overcome these problems by manually patching pod specs (unless the required intervention
prevents the pod to become available) but this is impractical, especially if this should be done frequently
and that's why, users first reaction is opening an issue about this or pull request proposing a new flag to
manage only a particular fields in pod specs. Adding a new flag for every field puts a debug command at risk of
unmanageable and unmaintainable state from not only maintainers point of view but also users. 

Due to all these reasons, this proposal adds a custom profiling support on top of predefined profiles debug command.
Custom profiling mitigates the new flag request pressure.

### Goals

* Make kubectl debug pod/node or ephemeral container spec configurable.

### Non-Goals

* Change the functionality of how `kubectl debug` works

## Proposal

There will be a new flag, namely `custom`, in `kubectl debug` which is used to pass
a json file that includes the fields that are compatible with partial container spec (e.g.

```json
{
  "ports": [
    {
      "containerPort": 80
    }
  ],
  "resources": {
    "limits": {
      "cpu": "0.5",
      "memory": "512Mi"
    },
    "requests": {
      "cpu": "0.2",
      "memory": "256Mi"
    }
  },
  "env": [
    {
      "name": "ENV_VAR1",
      "value": "value1"
    },
    {
      "name": "ENV_VAR2",
      "value": "value2"
    }
  ]
}
```
)
It is expected that this file passed to `custom` flag is decodable to `corev1.Container`
(please note that this doesn't have to be a complete `corev1.Container` spec, but all the
fields should be mapped to the fields in `corev1.Container`), otherwise `kubectl debug` returns an error
mentioning that only `corev1.Container` compatible json files are accepted.

User can still continue using current profiles (general, restricted, baseline, etc.) and when custom 
profile is passed, custom profile json is patched onto the latest version of container spec generated by predefined
profiles. As a result, custom profiles always suppress the properties inside predefined profiles in cases
of conflicts. Because for example, user may pass security context via custom profiles and netadmin profile has its own
security context properties and custom profile should override it.

To achieve this patching (and overriding) mechanism, custom profiling uses `StrategicMergePatch` that has already been used in code base
and proves that it covers such cases. This is an example of code portion demonstrates that how SMP will be used;

```go
	patchedContainer, err := strategicpatch.StrategicMergePatch(debugContainerJS, customJS, corev1.Container{})
	if err != nil {
		return fmt.Errorf("error creating three way patch to add debug container: %v", err)
	}
``` 

This feature focuses only on container spec customization and has no attempt to change the other fields in pod spec. 
The reason of this decision is that there are three types of debugging methods(copy to node, copy to pod and ephemeral container) 
and their largest intersection type is corev1.Container. Copy to node and Copy to pod both cover corev1.PodSpec type as
opposed to ephemeral container which only manifests itself as corev1.Container.
Therefore, pod spec related changes can be managed by flags which presumably should be only a few (annotations, labels, etc.).

Besides that Container spec has several fields and to prevent possible confusions and start more restrictive, custom
profile does not allow some fields to be used to overwrite, such as Name, Command, Image, Lifecycle and VolumeDevices (
first 3 fields have their own flags already). We can extend disallowed fields as a starting point and consider enabling
more fields per request separately in the future.

### User Stories (Optional)

<!--
Detail the things that people will be able to do if this KEP is implemented.
Include as much detail as possible so that people can understand the "how" of
the system. The goal here is to make this feel real for users without getting
bogged down.
-->

#### Story 1

As a cluster administrator, I'd like to debug a pod that requires an environment
variable to work regardless of the profile I choose.

#### Story 2

As a network administrator, I'd like to debug a node that requires to mount a 
specific volume using netadmin profile.

#### Story 3

As a restricted user, I'd be able to debug a pod to simulate the exact environment
of the problematic pod which has resource requests and limits.

### Notes/Constraints/Caveats (Optional)

<!--
What are the caveats to the proposal?
What are some important details that didn't come across above?
Go in to as much detail as necessary here.
This might be a good place to talk about core concepts and how they relate.
-->
* When user mounts a persistent volume claim to the debug pod, this must not affect 
the actual pod's functionality. If such a feature or property exists in storage,
kubectl debug should handle this and prevent mounting.
* kubectl debug pod --copy-to already copies the fields in original pod spec to
the copied pod spec which seems justifying the custom profiling. But in cases where
user may want to make changes on copied pod which is a different value than the
original, this custom profiling would also work for that purpose.

### Risks and Mitigations

<!--
What are the risks of this proposal, and how do we mitigate? Think broadly.
For example, consider both security and how this will impact the larger
Kubernetes ecosystem.

How will security be reviewed, and by whom?

How will UX be reviewed, and by whom?

Consider including folks who also work outside the SIG or subproject.
-->
* Unauthorized users may test the privileges by trying to mount 
a volume or use a port which is different from the application's, etc.
But that shouldn't be a problem because API server should reject the request.
If user has no permission to mount a PV, then this will be rejected.

## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs (though not always
required) or even code snippets. If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.
-->

This is the prospective code snippet of a function that applies the custom profile.
```go
func (o *DebugOptions) applyCustomProfile(debugPod *corev1.Pod, containerName string, ephemeral bool) error {
        // Find container in pod and encode it as json

        patchedContainer, err := strategicpatch.StrategicMergePatch(debugContainerJS, customJS, corev1.Container{})
        if err != nil {
            return fmt.Errorf("error creating three way patch to add debug container: %v", err)
        }

        err = json.Unmarshal(patchedContainer, &debugPod.Spec.Containers[index])
        if err != nil {
            return fmt.Errorf("unable to unmarshall patched container to ephemeral container: %v", err)
        }
        ...
    }
```

After predefined profiles are applied, there will be a generated podSpec that is pending for creation.
Before sending a request to API server to create this pod or ephemeral container, `applyCustomProfile` function will be called
with the required parameters. This function modifies podSpec internally and this podSpec will be created/updated.

### Test Plan

<!--
**Note:** *Not required until targeted at a release.*
The goal is to ensure that we don't accept enhancements with inadequate testing.

All code is expected to have adequate tests (eventually with coverage
expectations). Please adhere to the [Kubernetes testing guidelines][testing-guidelines]
when drafting this test plan.

[testing-guidelines]: https://git.k8s.io/community/contributors/devel/sig-testing/testing.md
-->

[x] I/we understand the owners of the involved components may require updates to
existing tests to make this code solid enough prior to committing the changes necessary
to implement this enhancement.

##### Prerequisite testing updates

<!--
Based on reviewers feedback describe what additional tests need to be added prior
implementing this enhancement to ensure the enhancements have also solid foundations.
-->

##### Unit tests

<!--
In principle every added code should have complete unit test coverage, so providing
the exact set of tests will not bring additional value.
However, if complete unit test coverage is not possible, explain the reason of it
together with explanation why this is acceptable.
-->

<!--
Additionally, for Alpha try to enumerate the core package you will be touching
to implement this enhancement and provide the current unit coverage for those
in the form of:
- <package>: <date> - <current test coverage>
The data can be easily read from:
https://testgrid.k8s.io/sig-testing-canaries#ci-kubernetes-coverage-unit

This can inform certain test coverage improvements that we want to do before
extending the production code to implement this enhancement.
-->

- `k8s.io/kubernetes/vendor/k8s.io/kubectl/pkg/cmd/debug`: `13-10-2023` - `62.7%`

New unit test cases will be added for the following scenarios:
- When custom profile is set, custom profile does not modify container name, command, image
- During the ephemeral container debugging, custom profile is set and `--profile` is `netadmin` and the pod output is expected
- During the ephemeral container debugging, Custom profile is set and `--profile` is `general` and the output is expected
- During the copying pod, custom profile is set and `--profile` is `general` and the output is expected
- During the node debugging, custom profile is set and `--profile` is `general` and the output is expected

- `k8s.io/kubernetes/vendor/k8s.io/kubectl/pkg/cmd/debug`: `30-09-2024` - `67.3%`

##### Integration tests

<!--
Integration tests are contained in k8s.io/kubernetes/test/integration.
Integration tests allow control of the configuration parameters used to start the binaries under test.
This is different from e2e tests which do not allow configuration of parameters.
Doing this allows testing non-default options and multiple different and potentially conflicting command line options.
-->

<!--
This question should be filled when targeting a release.
For Alpha, describe what tests will be added to ensure proper quality of the enhancement.

For Beta and GA, add links to added tests together with links to k8s-triage for those tests:
https://storage.googleapis.com/k8s-triage/index.html
-->

- Prepare a custom profile json including the fields in predefined profiles with different value and assure that the
custom profile's value supersedes the value in predefined profile.
- Prepare a custom profile json including the fields in pod spec with different value and assure that the custom
profile's value supersedes the value in original pod spec.
- Prepare a custom profile json including a new field not existed in pod spec or predefined profiles and assure that
this value is in the pod spec.
- Send invalid custom profile json(not in corev1.Container type or completely invalid json) and assure that
the error message is correct.

integration tests (defined in https://k8s.io/kubernetes/test/cmd/debug.sh#L571-L661) are running in 
https://storage.googleapis.com/k8s-triage/index.html?pr=1&job=pull-kubernetes-integration


##### e2e tests

<!--
This question should be filled when targeting a release.
For Alpha, describe what tests will be added to ensure proper quality of the enhancement.

For Beta and GA, add links to added tests together with links to k8s-triage for those tests:
https://storage.googleapis.com/k8s-triage/index.html

We expect no non-infra related flakes in the last month as a GA graduation criteria.
-->

- Prepare a custom profile and after applying it to live cluster, assure that pod spec's values are expected and
debugging works properly.

### Graduation Criteria

<!--
**Note:** *Not required until targeted at a release.*

Define graduation milestones.

These may be defined in terms of API maturity, [feature gate] graduations, or as
something else. The KEP should keep this high-level with a focus on what
signals will be looked at to determine graduation.

Consider the following in developing the graduation criteria for this enhancement:
- [Maturity levels (`alpha`, `beta`, `stable`)][maturity-levels]
- [Feature gate][feature gate] lifecycle
- [Deprecation policy][deprecation-policy]

Clearly define what graduation means by either linking to the [API doc
definition](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-versioning)
or by redefining what graduation means.

In general we try to use the same stages (alpha, beta, GA), regardless of how the
functionality is accessed.

[feature gate]: https://git.k8s.io/community/contributors/devel/sig-architecture/feature-gates.md
[maturity-levels]: https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions
[deprecation-policy]: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

Below are some examples to consider, in addition to the aforementioned [maturity levels][maturity-levels].

#### Alpha

- Feature implemented behind a feature flag
- Initial e2e tests completed and enabled

#### Beta

- Gather feedback from developers and surveys
- Complete features A, B, C
- Additional tests are in Testgrid and linked in KEP

#### GA

- N examples of real-world usage
- N installs
- More rigorous forms of testing—e.g., downgrade tests and scalability tests
- Allowing time for feedback

**Note:** Generally we also wait at least two releases between beta and
GA/stable, because there's no opportunity for user feedback, or even bug reports,
in back-to-back releases.

**For non-optional features moving to GA, the graduation criteria must include
[conformance tests].**

[conformance tests]: https://git.k8s.io/community/contributors/devel/sig-architecture/conformance-tests.md

#### Deprecation

- Announce deprecation and support policy of the existing flag
- Two versions passed since introducing the functionality that deprecates the flag (to address version skew)
- Address feedback on usage/changed behavior, provided on GitHub issues
- Deprecate the flag
-->

#### Alpha

- Feature hidden behind an environment variable in kubectl.
- Unit tests are implemented and enabled.

#### Beta

- Gather feedback from developers and surveys.
- Environment variable is enabled by default and feature can be disabled explicitly.
- Integration tests are implemented and enabled.
- YAML formatted custom profile support is added

#### GA

- Feature gate (i.e. `KUBECTL_DEBUG_CUSTOM_PROFILE`) is locked to true and will be removed in 1.34.
- e2e tests are implemented and enabled.

### Upgrade / Downgrade Strategy

<!--
If applicable, how will the component be upgraded and downgraded? Make sure
this is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to maintain previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to make use of the enhancement?
-->

NA

### Version Skew Strategy

<!--
If applicable, how will the component handle version skew with other
components? What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:
- Does this enhancement involve coordinating behavior in the control plane and nodes?
- How does an n-3 kubelet or kube-proxy without this feature available behave when this feature is used?
- How does an n-1 kube-controller-manager or kube-scheduler without this feature available behave when this feature is used?
- Will any other components on the node change? For example, changes to CSI,
  CRI or CNI may require updating that component before the kubelet.
-->

Copying pod and node debugging use built-in API endpoints. Ephemeral container functionality was
promoted to stable in 1.25. Besides custom profiling feature only relies on
`corev1.Container` type's existence differently from `kubectl debug` which is also built-in.
It is zero probability that this feature touches unavailable API endpoints in regard to
version skew strategy.

## Production Readiness Review Questionnaire

<!--

Production readiness reviews are intended to ensure that features merging into
Kubernetes are observable, scalable and supportable; can be safely operated in
production environments, and can be disabled or rolled back in the event they
cause increased failures in production. See more in the PRR KEP at
https://git.k8s.io/enhancements/keps/sig-architecture/1194-prod-readiness.

The production readiness review questionnaire must be completed and approved
for the KEP to move to `implementable` status and be included in the release.

In some cases, the questions below should also have answers in `kep.yaml`. This
is to enable automation to verify the presence of the review, and to reduce review
burden and latency.

The KEP must have a approver from the
[`prod-readiness-approvers`](http://git.k8s.io/enhancements/OWNERS_ALIASES)
team. Please reach out on the
[#prod-readiness](https://kubernetes.slack.com/archives/CPNHUMN74) channel if
you need any help or guidance.
-->

### Feature Enablement and Rollback

<!--
This section must be completed when targeting alpha to a release.
-->

This feature in alpha phase will be hidden behind `KUBECTL_DEBUG_CUSTOM_PROFILE`
environment variable and new flag can only be seen when this flag is exported
`export KUBECTL_DEBUG_CUSTOM_PROFILE=true`. User can easily disable this feature
via unsetting this environment variable.

###### How can this feature be enabled / disabled in a live cluster?

<!--
Pick one of these and delete the rest.

Documentation is available on [feature gate lifecycle] and expectations, as
well as the [existing list] of feature gates.

[feature gate lifecycle]: https://git.k8s.io/community/contributors/devel/sig-architecture/feature-gates.md
[existing list]: https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/
-->

- [ ] Feature gate (also fill in values in `kep.yaml`)
  - Feature gate name:
  - Components depending on the feature gate:
- [X] Other
  - Describe the mechanism: export KUBECTL_DEBUG_CUSTOM_PROFILE=true
  - Will enabling / disabling the feature require downtime of the control
    plane? No
  - Will enabling / disabling the feature require downtime or reprovisioning
    of a node? No

###### Does enabling the feature change any default behavior?

<!--
Any change of default behavior may be surprising to users or break existing
automations, so be extremely careful here.
-->
Enabling `KUBECTL_DEBUG_CUSTOM_PROFILE` environment variable will enable a new `custom` flag
in `kubectl debug`.

###### Can the feature be disabled once it has been enabled (i.e. can we roll back the enablement)?

<!--
Describe the consequences on existing workloads (e.g., if this is a runtime
feature, can it break the existing applications?).

Feature gates are typically disabled by setting the flag to `false` and
restarting the component. No other changes should be necessary to disable the
feature.

NOTE: Also set `disable-supported` to `true` or `false` in `kep.yaml`.
-->
Yes, the way of enabling this feature is exporting `KUBECTL_DEBUG_CUSTOM_PROFILE=true`.
The way of disabling is simply unsetting this environment.

###### What happens if we reenable the feature if it was previously rolled back?
Flag becomes visible and no risk for the cluster.

###### Are there any tests for feature enablement/disablement?

<!--
The e2e framework does not currently support enabling or disabling feature
gates. However, unit tests in each component dealing with managing data, created
with and without the feature, are necessary. At the very least, think about
conversion tests if API types are being modified.

Additionally, for features that are introducing a new API field, unit tests that
are exercising the `switch` of feature gate itself (what happens if I disable a
feature gate after having objects written with the new field) are also critical.
You can take a look at one potential example of such test in:
https://github.com/kubernetes/kubernetes/pull/97058/files#diff-7826f7adbc1996a05ab52e3f5f02429e94b68ce6bce0dc534d1be636154fded3R246-R282
-->
Enablement and disablement of this feature is managed by `KUBECTL_DEBUG_CUSTOM_PROFILE` environment
variable. When user sets this environment variable, new `custom` flag becomes visible. We can
add basic enablement/disablement test to check that flag is visible or not.

### Rollout, Upgrade and Rollback Planning

<!--
This section must be completed when targeting beta to a release.
-->

###### How can a rollout or rollback fail? Can it impact already running workloads?

<!--
Try to be as paranoid as possible - e.g., what if some components will restart
mid-rollout?

Be sure to consider highly-available clusters, where, for example,
feature flags will be enabled on some API servers and not others during the
rollout. Similarly, consider large clusters and how enablement/disablement
will rollout across nodes.
-->

User may pass a configmap(or any other that is used by other components) in custom profiling
and during debugging, user may modify (intentionally or unintentionally) these resources in an unexpected way for
the workloads. But this scenario is not different from any simple pod creation. Since there
is no scenario impacting the workloads specific to this feature. My answer is no to this question.

###### What specific metrics should inform a rollback?

<!--
What signals should users be paying attention to when the feature is young
that might indicate a serious problem?
-->
NA

###### Were upgrade and rollback tested? Was the upgrade->downgrade->upgrade path tested?

<!--
Describe manual testing that was done and the outcomes.
Longer term, we may want to require automated upgrade/rollback tests, but we
are missing a bunch of machinery and tooling and can't do that now.
-->
NA

###### Is the rollout accompanied by any deprecations and/or removals of features, APIs, fields of API types, flags, etc.?

<!--
Even if applying deprecation policies, they may still surprise some users.
-->
NA

### Monitoring Requirements

<!--
This section must be completed when targeting beta to a release.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.
-->
NA

###### How can an operator determine if the feature is in use by workloads?

<!--
Ideally, this should be a metric. Operations against the Kubernetes API (e.g.,
checking if there are objects with field X set) may be a last resort. Avoid
logs or events for this purpose.
-->
Users can determine by checking the value of `KUBECTL_DEBUG_CUSTOM_PROFILE` environment variable.
However, from the cluster admin point of view, all the APIs that this feature use have already been GA-ed,
so that it is hard to distinguish whether users enable this feature on their locals after exporting
the feature environment variable.

###### How can someone using this feature know that it is working for their instance?

<!--
For instance, if this is a pod-related feature, it should be possible to determine if the feature is functioning properly
for each individual pod.
Pick one more of these and delete the rest.
Please describe all items visible to end users below with sufficient detail so that they can verify correct enablement
and operation of this feature.
Recall that end users cannot usually observe component logs or access metrics.
-->

- [ ] Events
  - Event Reason: 
- [ ] API .status
  - Condition name: 
  - Other field: 
- [X] Other (treat as last resort)
  - Details: Checking the value of `KUBECTL_DEBUG_CUSTOM_PROFILE` environment variable, or
    they can run a test debug and see if their profile is respected in the resulting container.

###### What are the reasonable SLOs (Service Level Objectives) for the enhancement?

<!--
This is your opportunity to define what "normal" quality of service looks like
for a feature.

It's impossible to provide comprehensive guidance, but at the very
high level (needs more precise definitions) those may be things like:
  - per-day percentage of API calls finishing with 5XX errors <= 1%
  - 99% percentile over day of absolute value from (job creation time minus expected
    job creation time) for cron job <= 10%
  - 99.9% of /health requests per day finish with 200 code

These goals will help you determine what you need to measure (SLIs) in the next
question.
-->
NA

###### What are the SLIs (Service Level Indicators) an operator can use to determine the health of the service?

<!--
Pick one more of these and delete the rest.
-->

- [ ] Metrics
  - Metric name:
  - [Optional] Aggregation method:
  - Components exposing the metric:
- [X] Other (treat as last resort)
  - Details: Not applicable

###### Are there any missing metrics that would be useful to have to improve observability of this feature?

<!--
Describe the metrics themselves and the reasons why they weren't added (e.g., cost,
implementation difficulties, etc.).
-->
No

### Dependencies

<!--
This section must be completed when targeting beta to a release.
-->

###### Does this feature depend on any specific services running in the cluster?

<!--
Think about both cluster-level services (e.g. metrics-server) as well
as node-level agents (e.g. specific version of CRI). Focus on external or
optional services that are needed. For example, if this feature depends on
a cloud provider API, or upon an external software-defined storage or network
control plane.

For each of these, fill in the following—thinking about running existing user workloads
and creating new ones, as well as about cluster-level services (e.g. DNS):
  - [Dependency name]
    - Usage description:
      - Impact of its outage on the feature:
      - Impact of its degraded performance or high-error rates on the feature:
-->
No

### Scalability

<!--
For alpha, this section is encouraged: reviewers should consider these questions
and attempt to answer them.

For beta, this section is required: reviewers must answer these questions.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.
-->

###### Will enabling / using this feature result in any new API calls?

<!--
Describe them, providing:
  - API call type (e.g. PATCH pods)
  - estimated throughput
  - originating component(s) (e.g. Kubelet, Feature-X-controller)
Focusing mostly on:
  - components listing and/or watching resources they didn't before
  - API calls that may be triggered by changes of some Kubernetes resources
    (e.g. update of object X triggers new updates of object Y)
  - periodic API calls to reconcile state (e.g. periodic fetching state,
    heartbeats, leader election, etc.)
-->
No, SMP patching only happens on client side and there is no additional request to API server.

###### Will enabling / using this feature result in introducing new API types?

<!--
Describe them, providing:
  - API type
  - Supported number of objects per cluster
  - Supported number of objects per namespace (for namespace-scoped objects)
-->
No

###### Will enabling / using this feature result in any new calls to the cloud provider?

<!--
Describe them, providing:
  - Which API(s):
  - Estimated increase:
-->
No

###### Will enabling / using this feature result in increasing size or count of the existing API objects?

<!--
Describe them, providing:
  - API type(s):
  - Estimated increase in size: (e.g., new annotation of size 32B)
  - Estimated amount of new objects: (e.g., new Object X for every existing Pod)
-->
This may slightly increase the size of copied debugging pod according to the users custom profile spec or
debugging is performed via ephemeral container, this will slightly increase the pod size.

###### Will enabling / using this feature result in increasing time taken by any operations covered by existing SLIs/SLOs?

<!--
Look at the [existing SLIs/SLOs].

Think about adding additional work or introducing new steps in between
(e.g. need to do X to start a container), etc. Please describe the details.

[existing SLIs/SLOs]: https://git.k8s.io/community/sig-scalability/slos/slos.md#kubernetes-slisslos
-->
No

###### Will enabling / using this feature result in non-negligible increase of resource usage (CPU, RAM, disk, IO, ...) in any components?

<!--
Things to keep in mind include: additional in-memory state, additional
non-trivial computations, excessive access to disks (including increased log
volume), significant amount of data sent and/or received over network, etc.
This through this both in small and large cases, again with respect to the
[supported limits].

[supported limits]: https://git.k8s.io/community//sig-scalability/configs-and-limits/thresholds.md
-->
No

###### Can enabling / using this feature result in resource exhaustion of some node resources (PIDs, sockets, inodes, etc.)?

<!--
Focus not just on happy cases, but primarily on more pathological cases
(e.g. probes taking a minute instead of milliseconds, failed pods consuming resources, etc.).
If any of the resources can be exhausted, how this is mitigated with the existing limits
(e.g. pods per node) or new limits added by this KEP?

Are there any tests that were run/should be run to understand performance characteristics better
and validate the declared limits?
-->
No

### Troubleshooting

<!--
This section must be completed when targeting beta to a release.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.

The Troubleshooting section currently serves the `Playbook` role. We may consider
splitting it into a dedicated `Playbook` document (potentially with some monitoring
details). For now, we leave it here.
-->

###### How does this feature react if the API server and/or etcd is unavailable?

Custom profiling will happen on client side and since there is no change in `kubectl debug` functionality,
debugging will fail because pod will not be created.

###### What are other known failure modes?

<!--
For each of them, fill in the following information by copying the below template:
  - [Failure mode brief description]
    - Detection: How can it be detected via metrics? Stated another way:
      how can an operator troubleshoot without logging into a master or worker node?
    - Mitigations: What can be done to stop the bleeding, especially for already
      running user workloads?
    - Diagnostics: What are the useful log messages and their required logging
      levels that could help debug the issue?
      Not required until feature graduated to beta.
    - Testing: Are there any tests for failure mode? If not, describe why.
-->

- invalid debug pod after invalid custom profiling
  - Detection: Debug pod is not in running state or not having required privileges to debug smoothly
  - Mitigations: Pod can be deleted and re-run after modifying the custom profile
  - Diagnostics: Pod can't be attached or in not running state
  - Testing: There are several unit tests are implemented in https://github.com/kubernetes/kubectl/blob/master/pkg/cmd/debug/debug_test.go

###### What steps should be taken if SLOs are not being met to determine the problem?

Not applicable

## Implementation History

<!--
Major milestones in the lifecycle of a KEP should be tracked in this section.
Major milestones might include:
- the `Summary` and `Motivation` sections being merged, signaling SIG acceptance
- the `Proposal` section being merged, signaling agreement on a proposed design
- the date implementation started
- the first Kubernetes release where an initial version of the KEP was available
- the version of Kubernetes where the KEP graduated to general availability
- when the KEP was retired or superseded
-->
- 2023-10-13: Kep is proposed as alpha feature
- 2024-06-04: Kep is promoted to beta
- 2024-09-05: Kep is promoted to stable
- 2025-01-20: Kep is marked as implemented

## Drawbacks

<!--
Why should this KEP _not_ be implemented?
-->

## Alternatives

<!--
What other approaches did you consider, and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

### Flags for all fields
The alternative solution would be providing a flag in kubectl debug for each field. For example,
if user wants to mount a volume, there will be `--mount-volume` flag and user explicitly 
specifies the volume. However, this has a major drawback that it results in kubectl debug command
falling into unmanageable category with a numerous flags and users don't understand which one should be
used and for all new fields in pod spec, there is pressure to create a new flag in kubectl debug to support.

### Customizable Pod Spec instead corev1.Container
The alternative solution would be instead of custom profile json accepts corev1.Container type,
it will get PodSpec template and this covers the need of customizing all fields in Pod Spec including the labels 
and annotations. But this isn't applicable for ephemeral containers because they are residing in the 
original pod rather than copied pod.