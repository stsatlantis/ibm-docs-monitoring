---

copyright:
  years:  2018, 2023
lastupdated: "2023-09-11"

keywords: IBM Cloud, monitoring, regions

subcollection: monitoring

---

{{site.data.keyword.attribute-definition-list}}


# Regions
{: #regions}

A list of supported regions for the {{site.data.keyword.mon_full_notm}} service.
{: shortdesc}

The {{site.data.keyword.mon_full_notm}} service is available in the following regions:

![The image shows the locations where the {{site.data.keyword.mon_full_notm}} service is available.](images/Cloud-Monitoring-02-Locations.svg){: caption="Figure 1. Displays the regions where you can create and manage {{site.data.keyword.mon_full_notm}} resources." caption-side="bottom"}

This image is an artistic representation and does not reflect actual political or geographic boundaries.
{: note}

You can create {{site.data.keyword.mon_full_notm}} resources in one of the supported {{site.data.keyword.cloud_notm}} locations, which represent the geographic area where your {{site.data.keyword.mon_full_notm}} requests are handled and processed.


The following table lists the locations where the service is available:

| Geography             | Region                   | EU-Supported | HA Status |
|-----------------------|--------------------------|--------------|-----------|
| `Asia Pacific`        | `Sydney (au-syd)`        | `N/A`        | `MZR`     |
| `Asia Pacific`        | `Osaka (jp-osa)`         | `N/A`        | `N/A`     |
| `Asia Pacific`        | `Tokyo (jp-tok)`         | `N/A`        | `MZR`     |
| `Europe`              | `Frankfurt (eu-de) (*)`  | `YES`        | `MZR`     |
| `Europe`              | `London (eu-gb)`         | `NO`         | `MZR`     |
| `Europe`              | `Madrid (eu-es) (*) (**)`     | `YES`        | `MZR`     |
| `North America`       | `Dallas (us-south)`      | `N/A`        | `MZR`     |
| `North America`       | `Toronto (ca-tor)`       | `N/A`        | `MZR`     |
| `North America`       | `Washington (us-east)`   | `N/A`        | `MZR`     |
| `South America`       | `Sao Paulo (br-sao)`     | `N/A`        | `MZR`     |
{: caption="Table 1. List of locations where the service is available" caption-side="top"}

Where
* A *geography* is a geographic area or larger political body that contains one or more regions.
* A *region* is a defined geographic territory. A region could be a specific postal code area, a town, a city, a state, a group of states, or even a group of countries.
* `N/A` means feature that is not applicable to that geography.

`(*)` For more information, see [Enabling EU support for your account](/docs/account?topic=account-eu-supported).

`(**)` The [`Graduated Tier - Sysdig Secure + Monitor` plan](/docs/monitoring?topic=monitoring-service_plans) is not available in the Madrid (eu-es) region.
