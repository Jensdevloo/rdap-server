---
title: Overriding Red Dog Queries
---

# Overriding Red Dog Queries

## Introduction

This document is a formal definition of the SQL Provider’s query interface. Users that aim to implement [Option 2](intro.html#option-2-overriding-sql-provider-queries) need to provide queries that fulfill these requirements.

## How to override queries

The queries are expected to be found in a directory called `user_sql_files/`. This directory should be a collection of `.sql` files, mirroring the ones from the [default implementation](https://github.com/NICMx/rdap-sql-provider/tree/master/src/main/resources/META-INF/sql). The [installation document](server-install-option-2.html) states where this directory should be placed to be detected by Red Dog.

Except for queries referred as “catalogs”, isn't expected to override all the objects, and within each object isn't required to define every query. Only the necessary queries should be identified and provided. Red Dog will return an [HTTP 501 status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#501) if it receives a request whose result information depends of an undefined query.

Each `.sql` file should list the queries needed to interact with the object. The following rules must be followed by each query:
* Name the query: the first line must contain the name, which is declared using a '#' character followed by a string (eg. #getByRange).
* Parameters must be represented with the character '?', since [PreparedStatement](https://docs.oracle.com/javase/8/docs/api/java/sql/PreparedStatement.html) objects are used by the implementation. 
* The query MUST end with a semi-colon ';', since this is the expected delimiter per query definition.
* Optionally, the DB schema for each query can be parameterized using the string `{schema}`. This parameter will be replaced with the default value defined at [sql_provider_configuration.properties](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql_provider_configuration.properties), or with the custom value at [data-access.properties](https://github.com/NICMx/rdap-server/blob/master/src/main/webapp/WEB-INF/data-access.properties#L2). 

As an example, here’s how one might implement `Autnum.sql`:

```sql
	#getByRange
	SELECT asn.asn_id, asn.asn_handle, asn.asn_start_autnum, asn.asn_end_autnum, asn.asn_name, asn.asn_type, asn.asn_port43, asn.ccd_id 
	FROM {schema}.autonomous_system_number asn 
	WHERE asn.asn_start_autnum <= ? AND asn.asn_end_autnum >= ?;

	#getAutnumByEntity
	SELECT asn.asn_id, asn.asn_handle, asn.asn_start_autnum, asn.asn_end_autnum, asn.asn_name, asn.asn_type, asn.asn_port43, asn.ccd_id 
	FROM {schema}.autonomous_system_number asn 
	JOIN {schema}.asn_entity_roles ent ON ent.asn_id = asn.asn_id 
	WHERE ent.ent_id = ?;
```

The names of the columns and the number and order of parameters (‘?’ symbols) need to match the requirements listed for each query in the following sections. It should be noted that SQL aliases can be used to rename columns when the query alone would yield some other name.

The Red Dog implemented SQL files are the following:

*	[Autnum.sql](#autnum-sql)
*	[Domain.sql](#domain-sql)
*	[DsData.sql](#dsdata-sql)
*	[Entity.sql](#entity-sql)
*	[Event.sql](#event-sql)
*	[IpAddress.sql](#ipaddress-sql)
*	[IpNetwork.sql](#ipnetwork-sql)
*	[KeyData.sql](#keydata-sql)
*	[Link.sql](#link-sql)
*	[Nameserver.sql](#nameserver-sql)
*	[PublicId.sql](#publicid-sql)
*	[RdapAccessRole.sql](#rdapaccessrole-sql)
*	[RdapUser.sql](#rdapuser-sql)
*	[Remark.sql](#remark-sql)
*	[RemarkDescription.sql](#remarkdescription-sql)
*	[SecureDNS.sql](#securedns-sql)
*	[Variant.sql](#variant-sql)
*	[VCard.sql](#vcard-sql)
*	[VCardPostalInfo.sql](#vcardpostalinfo-sql)
*	[Zone.sql](#zone-sql)

The Red Dog SQL files calling catalogs and other needed catalogs:
(If overwritten, the implementer MUST make sure all of these are set)

*	[CountryCode](#countrycode)
*	[EventAction](#eventaction)
*	[IpVersion](#ipversion)
*	[Roles](#roles)
*	[Status](#status)
*	[VariantRelation](#variantrelation)

## SQL Files

### Autnum.sql

This file loads an Autnum object which models Autonomous System number registrations found in RIRs.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|asn_id|	Long|	No|	ASN's id|	123|
|asn_handle|	String|	No|	A RIR-unique identifier of the autnum registration|	XXXXX|
|asn_start_autnum|	Long|	No|	Starting number in the block of Autonomous System numbers|	20|
|asn_end_autnum|	Long|	No|	Ending number in the block of Autonomous System numbers|	25|
|asn_name|	String|	Yes|	An identifier assigned to the autnum registration by the registration holder|	asn1324|
|asn_type|	String|	Yes|	A string containing a RIR-specific classification of the autnum|	public|
|asn_port43|	String|	Yes|	A string containing the fully qualified host name or IP address of the WHOIS server where the ASN instance may be found|whois.example.com|
|ccd_id|	Integer|	No|	Country code id (Refer to [CountryCode catalog](#countrycode))|	484|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Autnum.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Autnum.sql).


### Domain.sql

This file loads a Domain object that represents a DNS name and point of delegation. For RIRs, these delegation points are in the reverse DNS tree, whereas for DNRs, these delegation points are in the forward DNS tree.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|dom_id|	Long|	No|	Domain's id|	123|
|dom_handle|	String|	Yes|	An RIR/DNR unique identifier of the domain registration|	XXXX|
|dom_ldh_name|	String|	Yes|	A string representing a domain name in LDH form|	xn--exampl-gva|
|dom_unicode_name|	String|	Yes|	A string representing a domain in U-label form|	examplé|
|dom_port43|	String|	Yes|	A string containing the fully qualified host name or IP address of the WHOIS server where the domain instance may be found|whois.example.com|
|zone_id|	Integer|	No|	Zone's id (Refer to [Zone.sql](#zone-sql) as it is also needed)| 	1|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Domain.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Domain.sql).


### DsData.sql

This file loads a DSData data object. The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|dsd_id|	Long|	No|	Ds data's id|	123|
|sdns_id|	Long|	No|	Secure DNS's id (Refer to [SecureDNS.sql](#securedns-sql)|	123|
|dsd_keytag|	Integer|	No|	An integer as specified by the key tag field of a DNS DS record|	12345|
|dsd_algorithm|	Integer|	No|	An integer as specified by the algorithm field of a DNS DS record|	3|
|dsd_digest|	String|	No|	A string as specified by the digest field of a DNS DS record|	49FD46E6C4B45C55D4AC|
|dsd_digest_type|	Integer|	No|	An integer as specified by the digest type field of a DNS DS record|	1|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/DsData.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/DsData.sql).


### Entity.sql

This file loads an Entity object that represents the information of organizations, corporations, governments, non-profits, clubs, individual persons, and informal groups of people.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|ent_id|	Long|	No|	The entity's id assigned in the database|	123|
|ent_handle|	String|	Yes|	The entity's id assigned in the database|	XXXX|
|ent_port43|	String|	Yes|	The host or ip address of the WHOIS server where the entity instance may be found|	whois.example.com|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Entity.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Entity.sql).


### Event.sql

This file loads an Event object that represents events that have occurred on an object instance.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|eve_id|	Long|	No|	Event's id| 	123|
|eac_id|	Integer|	No|	Event's action's id. (Refer to [EventAction catalog](#eventaction))| 1|
|eve_actor|	String|	Yes|	Event actor|	XXXXX|
|eve_date|	Timestamp|	Yes|	Event date|	2017-12-31 23:59:59|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Event.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Event.sql).


### IpAddress.sql

This file loads the IP addresses of a Nameserver object.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|iad_id|Long|No|Ip address id|123|
|nse_id|Long|No|Nameserver's id (Refer to [Nameserver.sql](#nameserver-sql))|123|
|iad_type|Integer|No|	Ip address type (4 or 6)|4|
|iad_value|	String|	No|	Ip address decimal value|4204805978|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/IpAddress.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/IpAddress.sql).


### IpNetwork.sql

This file loads an IpNetwork object which models IP network registrations found in RIRs and contains information about the network registration and entities related to the IP network.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|ine_id|	Long|	No|	Ip network's id|	123|
|ine_handle|	String|	No|	An RIR/DNR unique identifier of the Ip network registration|	XXXXX|
|ine_start_address_up|	String|	Yes|	The up part of the starting IP address of the network|	2306144275399704592|
|ine_start_address_down|	String|	Yes|	The down part of the starting IP address of the network|	0|
|ine_end_address_up|	String|	Yes|	The up part of the ending IP address of the network|	2306144275399704592|
|ine_end_address_down|	String|	Yes|	The down part of the ending IP address of the network|	18446744073709551615|
|ine_name|	String|	Yes|	An identifier assigned to the network registration by the registration holder|	some_name|
|ine_type|	String|	Yes|	A string containing a RIR/DNR specific classification of the Network|	private|
|ine_port43|	String|	Yes|	A string containing the fully qualified host name or IP address of the WHOIS server where the Ip network instance may be found.	|whois.example.com|
|ccd_id|	Integer|	No|	Country code's id (Refer to [CountryCode catalog](#countrycode))|	484|
|ip_version_id|	Integer|	No|	Ip version's id (Refer to [IpVersion catalog](#ipversion))|	6|
|ine_parent_handle|	String|	Yes|	A string containing a RIR/DNR unique identifier of the parent network of this network registration|	XXXX|
|ine_cidr|	Integer|	Yes|	Network mask length of the IP address|	64|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/IpNetwork.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/IpNetwork.sql).


### KeyData.sql

This file loads the Key Data of a Secure DNS object.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|kd_id|	Long|	No|	Key Data's id|	123|
|kd_kd_sdns_id|	Long|	No|	Secure DNS id|	123|
|kd_flags|	Integer|	Yes|	Integer containing the flags|	256|
|kd_protocol|	Integer|	Yes|	Integer containing the protocol value|	3|
|kd_public_key|	String|	Yes|	Public Key Material|	105klfie05|
|kd_algorithm|	Integer|	Yes|	Public Key cryptographic algorithm|	5|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/KeyData.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/KeyData.sql).


### Link.sql

This file loads a Link used by an object.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|lin_id|	Long|	No|	Link id|	123|
|lin_value|	String|	Yes|	A string containing value field of a link|	http://example.net/ip/201.0.0.0/|
|lin_rel|	String|	Yes|	A string containing link relation|	self|
|lin_href|	String|	No|	A string containing the URL referred by the link|	http://example.net/ip/201.0.0.0/8|
|lin_title|	String|	Yes|	A string containing this link title|	title|
|lin_media|	String|	Yes|	A string containing the information style of the content of this link|	screen|
|lin_type|	String|	Yes|	A string containing the media type|	application/rdap+json|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Link.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Link.sql).


### Nameserver.sql

This file loads a Nameserver object which represents information regarding DNS nameservers used in both forward and reverse DNS.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|nse_id|	Long|	No|	Nameserver's id|	123|
|nse_handle|	String|	Yes|	A RIR/DNR unique identifier of the nameserver registration|	XXXXX|
|nse_ldh_name|	String|	Yes|	A string describing a nameserver name in LDH form as described| ns1.xn--exampl-gva.com|
|nse_unicode_name|	String|	Yes|	A string containing a nameserver name with U-labels|	ns1.examplé.com|
|nse_port43|	String|	Yes|	A simple string containing the fully qualified host name or IP address of the WHOIS server where the nameserver instance may be found|	whois.example.com|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Nameserver.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Nameserver.sql).


### PublicId.sql

This file loads the Public ID used by an Entity or Domain object.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|pid_id|	Long|	No|	Public id's id|	123|
|pid_type|	String|	Yes|	Public id's type|	IANA Registrar ID|
|pid_identifier|	String|	Yes|	Public id's identifier|	1705|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/PublicId.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/PublicId.sql).


### RdapAccessRole.sql

This file loads the Access Roles that a user may have to customize its access level.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|rar_name|	String|	No|	Role name|	FBI|
|rar_description|	String|	No|	Role description|	Federal Bureau of Investigation|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/RdapAccessRole.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/RdapAccessRole.sql).


### RdapUser.sql

This file loads a User that can authenticate to the application.

The following table describes each alias and value type that the queries must return as result columns:

|Alias Name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|rus_name|	String|	No|	User's name, unique|	user123|
|rus_pass|	String|	No|	User's password|	321resu|
|rus_max_search_results|	Integer|	Yes|	Max number of results that will be returned for the user|	100|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/RdapUser.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/RdapUser.sql).


### Remark.sql

This file loads a Remark that can be related to an object.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|rem_id|	Long|	No|	Remark's id|	123|
|rem_title|	String|	Yes|	Remark's title|	Title|
|rem_type|	String|	Yes|	Remark's type|	Advice|
|rem_lang|	String|	Yes|	Remark's language|	en|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Remark.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Remark.sql).


### RemarkDescription.sql

This file loads the descriptions that a Remark can have.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|rde_order|	Integer|	No|	Number showing placement of the description at the Remark|	3|
|rem_id|	Long|	No|	Remark's unique identifier (Refer to [Remark.sql](#remark-sql))|	123|
|rde_description|	String|	No|	Description content|	Description 3|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/RemarkDescription.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/RemarkDescription.sql).


### SecureDNS.sql

This file loads the general Secure DNS information related to a Domain object.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|sdns_id|	Long|	No|	Secure dns' id|	123|
|sdns_zone_signed|	Boolean|	No|	Flag to show if the zone has been signed (1=true, 0=false)|	1|
|sdns_delegation_signed|	Boolean|	No|	Flag to show if there are DS records in the parent (1=true, 0=false)|	1|
|sdns_max_sig_life|	Integer|	Yes|	An integer representing the signature lifetime in seconds to be used when creating the RRSIG DS record in the parent zone|	63000|
|dom_id|	Long|	No|	Related Domain id (Refer to [Domain.sql](#domain-sql))|	123|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/SecureDNS.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/SecureDNS.sql).


### Variant.sql

This file loads the Variants that a Domain object has, as well as its relation.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|var_id|	Long|	No|	Variant's id|	123|
|var_idn_table|	String|	Yes|	Variant's IDN table as listed at [IANA IDN TABLES](https://www.iana.org/domains/idn-tables)|	.lat Spanish|
|vna_ldh_name|	String|	No|	Variant name in LDH format|	xn--exampl-gva|
|vna_unicode_name|	String|	Yes|	Variant name in Unicode format.|	examplé|
|rel_id|	Integer|	No|	Variant Relation Id (Refer to [VariantRelation catalog](#variantrelation))|	1|
|dom_id|	Long|	No|	Related Domain id (Refer to [Domain.sql](#domain-sql))|	123|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Variant.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Variant.sql).


### VCard.sql 

This file loads the VCard object that can be related to another RDAP object.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|vca_id|	Long|	No|	Vcard's id|	123|
|vca_name|	String|	Yes|	Contact's name|	Joe Jobs|
|vca_company_name|	String|	Yes|	Contact's company name|	Orange|
|vca_company_url|	String|	Yes|	Contact's url|	http://www.orange.mx|
|vca_email|	String|	Yes|	Contact's email|	jj@orange.mx|
|vca_voice|	String|	Yes|	Contact's telephone|	81 8818181|
|vca_cellphone|	String|	Yes|	Contact's cellphone|	81 8181818181|
|vca_fax|	String|	Yes|	Contact's fax|	248.697.0908|
|vca_job_title|	String|	Yes|	Contact's job title|	Engineer|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/VCard.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/VCard.sql).


### VCardPostalInfo.sql

This file loads the VCard Postal Info related to a VCard object.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|vpi_id|	Long|	No|	Postal info's id|	123|
|vca_id|	String|	Yes|	Vcard's id (Refer to [VCard.sql](#vcard-sql))|	Joe Jobs|
|vpi_type|	String|	Yes|	Postal info's type|	local|
|vpi_country|	String|	Yes|	Country|	Mexico|
|vpi_city|	String|	Yes|	City|	Juarez|
|vpi_street1|	String|	Yes|	Street (first part)|	Luis Elizondo|
|vpi_street2|	String|	Yes|	Street (second part)|	Altavista|
|vpi_street3|	String|	Yes|	Street (third part)|	100|
|vpi_state|	String|	Yes|	State.|	Guadalajara|
|vpi_postal_code|	String|	Yes|	Postal code.|	34020|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/VCardPostalInfo.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/VCardPostalInfo.sql).


### Zone.sql

This file is extremely important for the [Domain object](#domain-sql) as all of the queries need this id to work.

The following table describes each alias and value type that the queries must return as result columns:

|Alias name|Value Type|Allows Null|Description|Example|
|:---------|:--------:|:---------:|:----------|:-----:|
|zone_id|	Integer|	No|	Zone id|	6|
|zone_name|	String|	No|	Zone name|	net|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Zone.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Zone.sql).


## Needed Catalogs

A custom implementation will need to add the following catalogs, as well as its queries so that Red Dog server will properly work.

### CountryCode

Red Dog uses the standard for area codes used by the [United Nations Statistics Division](https://en.wikipedia.org/wiki/United_Nations_Statistics_Division "United Nations Statistics Division from Wikipedia").The complete list of country codes can be found [here](http://www.nationsonline.org/oneworld/country_code_list.htm "ISO Alpha-2, Alpha-3, and Numeric Country Codes").

This catalog does not have a SQL file but is needed at DB since [Autnum.sql](#autnum-sql) and [IpNetwork.sql](#ipnetwork-sql) objects retrieve it.


### EventAction

The following values have been registered in the “RDAP JSON Values” registry for the EventAction catalog:

|EventAction name|	EventAction description|
|:--------------:|:------------------------|
|registration|	The object instance was initially registered.|
|reregistration|	The object instance was registered subsequently to initial registration.|
|last changed|	An action noting when the information in the object instance was last changed.|
|expiration|	The object instance has been removed or will be removed at a predetermined date and time from the registry.|
|deletion|	The object instance was removed from the registry at a point in time that was not predetermined.|
|reinstantiation|	The object instance was reregistered after having been removed from the registry.|
|transfer|	The object instance was transferred from one registrant to another.|
|locked|	The object instance was locked.|
|unlocked|	The object instance was unlocked.|

This catalog does not have a SQL file but is needed at DB since a [Event.sql](#event-sql) object retrieves it.


### IpVersion

The following values have been registered for the IpVersion catalog:

|IpVersion name|	IpVersion description|
|:--------------:|:------------------------|
|4|	IP v4|
|6|	IP v6|

This catalog does not have a SQL file but is needed at DB since a [IpNetwork.sql](#ipnetwork-sql) object retrieves it.


### Roles

The following values have been registered in the “RDAP JSON Values” registry for the Roles catalog:

|Role name|	Role description|
|:------:|:----------------|
|registrant|	The entity object instance is the registrant of the registration. In some registries, this is known as a maintainer.|
|technical|	The entity object instance is a technical contact for the registration.|
|administrative|	The entity object instance is an administrative contact for the registration.|
|abuse|	The entity object instance handles network abuse issues on behalf of the registrant of the registration.|
|billing|	The entity object instance handles payment and billing issues on behalf of the registrant of the registration.|
|registrar|	The entity object instance represents the authority responsible for the registration in the registry.|
|reseller|	The entity object instance represents a third party through which the registration was conducted (i.e., not the registry or registrar).|
|sponsor|	The entity object instance represents a domain policy sponsor, such as an ICANN-approved sponsor.|
|proxy|	The entity object instance represents a proxy for another entity object, such as a registrant.|
|notifications|	An entity object instance designated to receive notifications about association object instances.|
|noc|	The entity object instance handles communications related to a network operations center (NOC).|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Role.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Role.sql).

This catalog is needed so an object can have Entities with an associated role.


### Status

The following values have been registered in the “RDAP JSON Values” registry for the Status of the RDAP objects:

|RDAP Status name|	Status description|
|:--------------:|:-------------------|
|validated|	Signifies that the data of the object instance has been found to be accurate. This type of status is usually found on entity object instances to note the validity of identifying contact information.|
|renew prohibited|	Renewal or reregistration of the object instance is forbidden.|
|update prohibited|	Updates to the object instance are forbidden.|
|transfer prohibited|	Transfers of the registration from one registrar to another are forbidden.|
|delete prohibited|	Deletion of the registration of the object instance is forbidden.|
|proxy|	The registration of the object instance has been performed by a third party.|
|private|	The information of the object instance is not designated for public consumption.|
|removed|	Some of the information of the object instance has not been made available and has been removed.|
|obscured|	Some of the information of the object instance has been altered for the purposes of not readily revealing the actual information of the object instance.|
|associated|	The object instance is associated with other object instances in the registry.|
|active|	The object instance is in use. For domain names, it signifies that the domain name is published in DNS. For network and autnum registrations, it signifies that they are allocated or assigned for use in operational networks.|
|inactive|	The object instance is not in use.|
|locked|	Changes to the object instance cannot be made, including the association of other object instances.|
|pending create|	A request has been received for the creation of the object instance, but this action is not yet complete.|
|pending renew|	A request has been received for the renewal of the object instance, but this action is not yet complete.|
|pending transfer|	A request has been received for the transfer of the object instance, but this action is not yet complete.|
|pending update|	A request has been received for the update or modification of the object instance, but this action is not yet complete.|
|pending delete|	A request has been received for the deletion or removal of the object instance, but this action is not yet complete. For domains, this might mean that the name is no longer published in DNS but has not yet been purged from the registry database.|

The new SQL file must define the same queries and aliases as the Red Dog's implementation do, so both Red Dog's implementation and the own database columns coincide. The queries and aliases must be like [META-INF/sql/Status.sql](https://github.com/NICMx/rdap-sql-provider/blob/master/src/main/resources/META-INF/sql/Status.sql).

This catalog is needed since the following objects can retrieve it: [Autnum.sql](#autnum-sql), [Domain.sql](#domain-sql), [Entity.sql](#entity-sql), [IpNetwork.sql](#ipnetwork-sql), and [Nameserver.sql](#nameserver-sql).


### VariantRelation

The following values have been registered in the “RDAP JSON Values” registry for VariantRelations catalog:

|VariantRelation name|	VariantRelation description|
|:------------------:|:----------------------------|
|registered|	The variant names are registered in the registry.|
|unregistered|	The variant names are not found in the registry.|
|registration restricted|	Registration of the variant names is restricted to certain parties or within certain rules.|
|open registration|	Registration of the variant names is available to generally qualified registrants.|
|conjoined|	Registration of the variant names occurs automatically with the registration of the containing domain registration.|

This catalog does not have a SQL file but is needed at DB since a [Variant.sql](#variant-sql) object retrieves it.


## Additional Notes

*	All necessary values for the RDAP catalogs can be found [here](https://www.iana.org/assignments/rdap-json-values/rdap-json-values.xhtml#rdap-json-values-1 "RDAP JSON Values").
*	All the [provided queries](https://github.com/NICMx/rdap-sql-provider/tree/master/src/main/resources/META-INF/sql) are those implemented by **Red Dog**, if they are overwritten they don´t have to be the same, they only need to return the same type of data.

## Where to Go Next

After making your own queries now you will have to install the server, steps on how to do this can be found [here](server-install-option-2.html).
