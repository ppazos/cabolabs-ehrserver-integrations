# cabolabs-ehrserver-integrations

HL7 v2.x, FHIR, CDA integrations with openEHR for EHRServer

## About mappings between standards

Clinical data has many representations. Each representation has a very specific use,
and some of them have some level of overlapping. In general, most of the time in clinical
system and data integrations is spent on data mapping between different data representations
from different standards and local/custom models.

This repository includes work done as an exercise to know to:

1. Understand technical difficulties of different data mappings
2. Estimate the effort/cost/time on designing and implementing such mappings
3. Define a good tool set/platform for implementing and deploying mappings
4. Define a methodology for doing mappings


This experiment was done considering:

1. Inbound source formats from different standards/specifications
2. Mapping any source of data into a canonical model
3. The canonical model would be openEHR
4. For implementation purposes we'll like to store the transformed openEHR data into the EHRServer
5. The EHRServer data will be then used for data querying/analysis/extraction/visualization using the canonical model


Since we are familiar with the tools needed for system integration, we choose Mirth Connect
as the middleware between the source of the inbound messages/documents/resources and the
EHRServer Clinical Data Repository. Then mappings would be implemented in Mirth.


## Integrations

We choose three standards in the HL7 family to be the first set of source formats to
test our approach, measure effort, and create a repeatable methodology. Initially we
choose the domain of Laboratory Reports as our framework.

### FHIR integrations

Studying the FHIR specs, we found profiles for Lipids Panel, and some examples. We chose
one of the samples in JSON to be our first FHIR test case. Studied it's structure and
modified some structures, for instance the patient ID reference in the JSON resource
was transformed to a logical ID with the EHR UID of that patient, since the EHRServer
needs that data point to accept data.

Then we checked the openEHR CKM for archetypes that can model that specific structure
from FHIR, trying to comply with the semantics under the data. Found some draft archetypes
that were a perfect fit like the analyte archetype (openEHR-EHR-CLUSTER.laboratory_test_analyte.v0)
and the laboratory test result archetype (openEHR-EHR-OBSERVATION.laboratory_test_result.v0).
And the last part would be the report document that contains the OBSERVATION and the CLUSTER
(openEHR-EHR-COMPOSITION.report-result.v1).

The next step when we have all the archetypes is to create an openEHR template. The issue here
was the Template Designer doesn't allow to express certain constraints, like setting DV_TEXT
nodes (narrative text) into DV_CODED_TEXT (code+text), and we needed to express each result
in the Lipids Panel as a coded item. So we specialized the analyte archetype to explicitly have
a code, leading to the analyte coded archetype (openEHR-EHR-CLUSTER.laboratory_test_analyte-coded.v0).
This specialization was done using the Archetype Editor.

Then with the coded archetype, we created the template (Laboratory Result Report.oet) and
exported the Operational Template (laboratory_results_report.en.v1.opt). The OPT is what we
need to design the mapping, generate openEHR instances for testing, and load data into the EHRServer.

From the OPT we use the CaboLabs openEHR Toolkit (CoT) to generate an openEHR XML instance that
we can use as an example of the target format. Then we mapped node by node, value by value, from the
source FHIR JSON file, to the target openEHR XML file. Then we loaded both files into Mirth as
inbound and outbound file templates, and started to create transformations from the source into
the target, implementing the mappings, including some data transformations like date formats.

In Mirth Connect, we defined a HTTP Listener Source Connector that accepts JSON and sends XML,
and an HTTP Sender Destination Connector that accepts and sends XML. Then we defined a test
HTTP POST Request on Insomnia REST Client with the URL from the Mirth Channel Source Connector
and the FHIR JSON as payload.

To be able to receive the transformed data in EHRServer we need to:

1. upload the OPT into EHRServer
2. create an EHR, copy it's UID, and put it in the patient ID of the FHIR JSON in the Insomnia request
3. create an API Key for the REST API, so Mirth can commit data
4. setup an Authorization header with the API Key into the Mirth Channel Destination Connector
5. configure the rest of the request for the EHRServer REST API from Mirth

Finally after testing and fixing, we can hit the send button on Insomnia and receive different
Lipids Panels on EHRServer. With the data in EHRServer we can start creating queries and integrating
those queries into other applications.

This process will be tested for the other mappings to check if the same methodology can be
applied in a repeatable manner.

The effort to complete this whole working integration was:

1. About 1.5 days understanding the FHIR models, looking for samples, looking for archetypes, and planning next steps.
2. About 1 day on designing the mappings and testing technology.
3. About 1 day fixing archetypes, generating the operational template, generating openEHR XML instances, and validating everything.
4. About 1 day integrating everything together, implementing the mappings in Mirth, testing and enjoying the results.

Now with the experience of this first integration, I would say the next one will be much
faster, of course depending on the complexity of both source and target formats. For instance
now we have a more clear path on how to do this, and since we faced some technical issues that
were solved, we can avoid them in the future. These issues were mainly related with constraints
from openEHR modeling tools that are not being maintained, and the strange way in which the FHIR
resources are documented, since they invented a way of expressing schemas and constraints that is
not intuitive, nor standard.


### CDA integrations

TBD

### HL7 v2.x integrations

TBD

### Other integrations

If you are interested in having other integrations, please raise an issue here in
github explaining which formats do you want us to consider, and provide samples.
