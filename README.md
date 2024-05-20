KIP-1000: List Client Metrics Configuration Resources
================

Created by [Andrew
Schofield](https://cwiki.apache.org/confluence/display/~schofielaj),
last modified on [Nov 20,
2023](https://cwiki.apache.org/confluence/pages/diffpagesbyversion.action?pageId=278465373&selectedPageVersions=8&selectedPageVersions=9 "Show changes")

- [Status](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-Status)

- [Motivation](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-Motivation)

- [Proposed
  Changes](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-ProposedChanges)

- [Public
  Interfaces](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-PublicInterfaces)

  - [Kafka
    Protocol](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-KafkaProtocol)

    - [ListClientMetricsResources
      RPC](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-ListClientMetricsResourcesRPC)

  - [Admin
    Client](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-AdminClient)

  - [Tools](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-Tools)

    - [kafka-client-metrics.sh](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-kafka-client-metrics.sh)

- [Security](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-Security)

- [Compatibility, Deprecation, and Migration
  Plan](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-Compatibility,Deprecation,andMigrationPlan)

- [Test
  Plan](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-TestPlan)

- [Rejected
  Alternatives](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1000%3A+List+Client+Metrics+Configuration+Resources#KIP1000:ListClientMetricsConfigurationResources-RejectedAlternatives)

# Status

**Current state**: *Accepted*

**Discussion thread**:
[*here*](https://www.mail-archive.com/dev@kafka.apache.org/msg135343.html)*  
*

**JIRA**:
[*KAFKA-15831*](https://issues.apache.org/jira/browse/KAFKA-15831)*  
*

Please keep the discussion on the mailing list rather than commenting on
the wiki (wiki discussions get unwieldy fast).

# Motivation

This KIP introduces a way to list the client metrics configuration
resources introduced by KIP-714. The client metrics configuration
resources can be created, read, updated and deleted using the existing
RPCs and the `kafka-configs.sh` tool, in common with all of the other
configuration resources. There is, as yet, no way to list these
resources. Some configuration resources, such as brokers, are
singletons. Some are associated with other Kafka resources, such as
topics. Client metrics configuration resources are named, but not
associated with another Kafka resource, so there needs to be a way to
list them.

# Proposed Changes

The KIP adds a new RPC to the Kafka protocol called
`ListClientMetricsResources`  which responds with a list of the client
metrics configuration resources. It also adds
`AdminClient.listClientMetricsResources` to provide a programming
interface for building tools to list client metrics configuration
resources.

As an example, you could get the configuration for all of the client
metrics resources using two calls:

<table style="width:83%;">
<colgroup>
<col style="width: 83%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>AdminClient.listClientMetricsResources()</code></p>
<p><code>AdminClient.describeConfigs(Collection&lt;ConfigResource&gt;)</code></p></td>
</tr>
</tbody>
</table>

# Public Interfaces

This KIP introduces support for obtaining a list of client metrics
configuration resources to the Kafka protocol and admin client.

## Kafka Protocol

### ListClientMetricsResources RPC

Client metrics configuration is only supported for KRaft clusters. This
KIP is only supported for KRaft clusters.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>{</code></p>
<p><code>"apiKey": XX,</code></p>
<p><code>"type": "request",</code></p>
<p><code>"listeners": ["broker"],</code></p>
<p><code>"name": "ListClientMetricsResourcesRequest",</code></p>
<p><code>"validVersions": "0",</code></p>
<p><code>"flexibleVersions": "0+",</code></p>
<p><code>"fields": [</code></p>
<p><code>]</code></p>
<p><code>}</code></p>
<p><code>{</code></p>
<p><code>"apiKey": XX,</code></p>
<p><code>"type": "response",</code></p>
<p><code>"name": "ListClientMetricsResourcesResponse",</code></p>
<p><code>"validVersions": "0",</code></p>
<p><code>"flexibleVersions": "0+",</code></p>
<p><code>"fields": [</code></p>
<p><code>{ "name": "ThrottleTimeMs", "type": "int32", "versions": "0+",</code></p>
<p><code>"about": "The duration in milliseconds for which the request was throttled due to a quota violation, or zero if the request did not violate any quota."</code>
<code>},</code></p>
<p><code>{ "name": "ErrorCode", "type": "int16", "versions": "0+"</code>
<code>},</code></p>
<p><code>{ "name": "ClientMetricsResources", "type": "[]ClientMetricsResource", "versions": "0+", "fields": [</code></p>
<p><code>{ "name": "Name", "type": "string", "versions": "0+"</code>
<code>}</code></p>
<p><code>]}</code></p>
<p><code>]</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

## Admin Client

The following methods are added to the
`org.apache.kafka.client.admin.Admin` interface.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>/**</code></p>
<p><code>* List the client metrics configuration resources available in the cluster.</code></p>
<p><code>*</code></p>
<p><code>* @param options The options to use when listing the client metrics resources.</code></p>
<p><code>* @return The ListClientMetricsResourcesResult.</code></p>
<p><code>*/</code></p>
<p><code>ListClientMetricsResourcesResult listClientMetricsResources(ListClientMetricsResourcesOptions options);</code></p>
<p><code>/**</code></p>
<p><code>* List the client metrics configuration resources available in the cluster with the default options.</code></p>
<p><code>* &lt;p&gt;</code></p>
<p><code>* This is a convenience method for {@link #listClientMetricsResources(ListClientMetricsResourcesOptions)}</code></p>
<p><code>* with default options. See the overload for more details.</code></p>
<p><code>*</code></p>
<p><code>* @return The ListClientMetricsResourcesResult.</code></p>
<p><code>*/</code></p>
<p><code>default</code>
<code>ListClientMetricsResourcesResult listClientMetricsResources() {</code></p>
<p><code>return</code> <code>listClientMetricsResources(new</code>
<code>ListClientMetricsResourcesOptions());</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

The options are defined as follows:

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>package</code>
<code>org.apache.kafka.client.admin;</code></p>
<p><code>/**</code></p>
<p><code>* Options for {@link Admin#listClientMetricsResources()}.</code></p>
<p><code>*</code></p>
<p><code>* The API of this class is evolving, see {@link Admin} for details.</code></p>
<p><code>*/</code></p>
<p><code>@InterfaceStability.Evolving</code></p>
<p><code>public</code> <code>class</code>
<code>ListClientMetricsResourcesOptions extends</code>
<code>AbstractOptions&lt;ListClientMetricsResourcesOptions&gt; {</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

The result is defined as follows:

<table>
<colgroup>
<col style="width: 101%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>package</code>
<code>org.apache.kafka.clients.admin;</code></p>
<p><code>/**</code></p>
<p><code>* The result of the {@link Admin#listClientMetricsResources()} call.</code></p>
<p><code>* &lt;p&gt;</code></p>
<p><code>* The API of this class is evolving, see {@link Admin} for details.</code></p>
<p><code>*/</code></p>
<p><code>@InterfaceStability.Evolving</code></p>
<p><code>public</code> <code>class</code>
<code>ListClientMetricsResourcesResult {</code></p>
<p><code>/**</code></p>
<p><code>* Returns a future that yields either an exception, or the full set of client metrics</code></p>
<p><code>* resource listings.</code></p>
<p><code>*</code></p>
<p><code>* In the event of a failure, the future yields nothing but the first exception which</code></p>
<p><code>* occurred.</code></p>
<p><code>*/</code></p>
<p><code>public</code>
<code>KafkaFuture&lt;Collection&lt;ClientMetricsResourceListing&gt;&gt; all() {</code></p>
<p><code>}</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

And finally the listing itself, which is initially only the name:

<table style="width:79%;">
<colgroup>
<col style="width: 79%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>package</code>
<code>org.apache.kafka.clients.admin;</code></p>
<p><code>@InterfaceStability.Evolving</code></p>
<p><code>public</code> <code>class</code>
<code>ClientMetricsResourceListing {</code></p>
<p><code>private</code> <code>final</code> <code>String name;</code></p>
<p><code>public</code>
<code>ClientMetricsResourceListing(String name) {</code></p>
<p><code>this.name = name;</code></p>
<p><code>}</code></p>
<p><code>public</code> <code>String name() {</code></p>
<p><code>return</code> <code>name;</code></p>
<p><code>}</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

## Tools

### kafka-client-metrics.sh

A new option `--list`  is added to `kafka-client-metrics.sh` which lists
the client metrics resources without describing them.

|                                                                |
|----------------------------------------------------------------|
| `$ kafka-client-metrics.sh --bootstrap-server $BROKERS --list` |

# Security

Client metrics configuration resources are secured using the CLUSTER
resource.

|                            |         |                  |
|:---------------------------|:--------|:-----------------|
| ListClientMetricsResources | CLUSTER | DESCRIBE_CONFIGS |

# Compatibility, Deprecation, and Migration Plan

No impact.

# Test Plan

The KIP implementation will be accompanied by the usual set of unit
tests.

# Rejected Alternatives

None.
