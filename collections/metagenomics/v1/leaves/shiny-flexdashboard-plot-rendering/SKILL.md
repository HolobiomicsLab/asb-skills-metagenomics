---
name: shiny-flexdashboard-plot-rendering
description: Use when you need to serve and render interactive plots (epidemic curves,
  cluster networks) from cluster-labelled isolate metadata inside a Shiny/flexdashboard
  app, following the CATHAI deployment pattern.
when_to_use_negative:
- Input is not cluster-labelled isolate metadata (no collection dates or cluster assignments),
  so an EpiCurve-style temporal overview is meaningless
- You need interactive WGS cluster network exploration rather than temporal curve
  rendering — use the visNetwork cluster graph component instead of an epidemic curve
- 'The deployment cannot secure the Shiny server: CATHAI explicitly warns the shiny
  server must be bound to 127.0.0.1 or firewalled, so exposing it publicly is unsafe'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CATHAI
  role: the web-based visualization platform whose Shiny/flexdashboard layer renders
    the plots
  repo: https://github.com/FordeGenomics/cathai
- name: flexdashboard
  role: R dashboard framework hosting the rendered visualization layer
- name: Shiny
  role: R web framework serving the interactive plots behind the authentication portal
- name: EpiCurve
  role: R package that generates the epidemic curve plot, colour-coded by cluster
  repo: https://github.com/IamKDO/EpiCurve
- name: ISOweek
  role: bins isolate collection dates into ISO 8601 weeks for the epidemic curve x-axis
- name: ggplot2
  role: plotting layer under the epidemic curve rendering
- name: RColorBrewer
  role: supplies the palette used to colour epidemic curve bars by cluster
- name: visNetwork
  role: R package (vis.js) used for network visualization of WGS-informed clusters
    in the same dashboard
  repo: https://github.com/datastorm-open/visNetwork
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac040
    title: 'CATHAI: cluster analysis tool for healthcare-associated infections'
  source_content_hash: sha256:434a7e193b2c91f28b3bf5cd838cf213cccd9d68ea08958db292560273389a7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FordeGenomics/cathai
  tools:
  - CATHAI
  - EpiCurve
  - ISOweek
  - R
  - RColorBrewer
  - Shiny
  - flexdashboard
  - ggplot2
  - visNetwork
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# shiny-flexdashboard-plot-rendering

## Summary

Renders CATHAI's visualization layer — a flexdashboard/Shiny web application that displays WGS-informed cluster metadata as interactive plots such as cluster-coloured epidemic curves (EpiCurve over ISOweek-binned collection dates) and visNetwork cluster graphs. It is how the article turns cluster-labelled isolate tables into explorable outbreak surveillance graphics.

## When to use

Use when you need to serve and render interactive plots (epidemic curves, cluster networks) from cluster-labelled isolate metadata inside a Shiny/flexdashboard app, following the CATHAI deployment pattern.

## When NOT to use

- Input is not cluster-labelled isolate metadata (no collection dates or cluster assignments), so an EpiCurve-style temporal overview is meaningless
- You need interactive WGS cluster network exploration rather than temporal curve rendering — use the visNetwork cluster graph component instead of an epidemic curve
- The deployment cannot secure the Shiny server: CATHAI explicitly warns the shiny server must be bound to 127.0.0.1 or firewalled, so exposing it publicly is unsafe

## Inputs

- cluster-labelled isolate sample metadata table (isolate collection dates and cluster assignments) placed in the shiny/data directory
- CATHAI Docker image built from ./docker/Dockerfile

## Outputs

- rendered epidemic curve figure (e.g. epicurve.png) with bars binned by ISO week and colour-coded by cluster
- interactive Shiny/flexdashboard visualization page served on the proxied webapp port (8080:5000 in the documented run command)

## How to apply

Assemble the R stack CATHAI uses — flexdashboard, visNetwork, DT, shinyjs, ggplot2, ISOweek, EpiCurve, RColorBrewer, gghighlight, imager, bslib, shinydashboard — installed via install.packages with repos='https://cran.rstudio.com/'. Load the cluster-labelled isolate metadata (collection dates plus cluster assignments) from the shiny/data directory mounted into the container at /opt/cathai/shiny/data, bin collection dates into ISO 8601 weeks with ISOweek, and render the epidemic curve with EpiCurve, colour-coding bars by cluster using an RColorBrewer palette layered on ggplot2. Serve the plots through a Shiny server proxied behind the authentication dashboard portal (CATHAI uses shiny server to serve R shiny apps; run the webapp via the documented docker run command exposing port 8080:5000). Rationale: the epidemic curve gives a high-level overview of isolate distribution over time, and per-cluster colouring reveals cluster emergence and spread. Validate that the rendered figure (e.g. epicurve.png) exists and that bar colours correspond one-to-one with cluster labels in the metadata table.

## Related tools

- **CATHAI** (the web-based visualization platform whose Shiny/flexdashboard layer renders the plots) — https://github.com/FordeGenomics/cathai
- **flexdashboard** (R dashboard framework hosting the rendered visualization layer)
- **Shiny** (R web framework serving the interactive plots behind the authentication portal)
- **EpiCurve** (R package that generates the epidemic curve plot, colour-coded by cluster) — https://github.com/IamKDO/EpiCurve
- **ISOweek** (bins isolate collection dates into ISO 8601 weeks for the epidemic curve x-axis)
- **ggplot2** (plotting layer under the epidemic curve rendering)
- **RColorBrewer** (supplies the palette used to colour epidemic curve bars by cluster)
- **visNetwork** (R package (vis.js) used for network visualization of WGS-informed clusters in the same dashboard) — https://github.com/datastorm-open/visNetwork

## Examples

```
docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 run
```

## Evaluation signals

- The exported figure file (e.g. epicurve.png) exists in the output location
- Bar colours in the rendered epidemic curve correspond one-to-one with the cluster labels present in the metadata table
- Epidemic curve bars are binned by ISO 8601 week and counts over collection time match the isolate dates in the input metadata
- The webapp serves on the mapped port (docker run -p 8080:5000 ...) and the shiny server binds only to 127.0.0.1 or is firewalled
- All required R packages (flexdashboard, visNetwork, EpiCurve, ISOweek, RColorBrewer, ggplot2, etc.) install without error from the CRAN repo

## Limitations

- Shiny server must be bound to 127.0.0.1 or protected by a host firewall (local system or e.g. NeCTAR security group) to block shiny server ports from outside sources
- You must write your own config.py before deployment (see the sample config.py included in the repo)
- EpiCurve has known version-specific bugs (e.g. 2.4-2 fixed wrong x-axis labels when an hourly curve crosses a month); the CATHAI workflow uses weekly ISO week binning rather than hourly binning

## Evidence

- [methods] article methods R package install: [evidence span withheld — anchor status: not_found]
- [abstract] abstract platform description: "CATHAI (cluster analysis tool for healthcare-associated infections) is an interactive web-based visualization platform that couples WGS informed clustering with associated metadata"
- [readme] readme shiny server note: "* Uses shiny server to serve the R shiny apps that are proxied through the authenication dashboard portal"
- [readme] readme security note: "Make sure that the shiny server is only bound on 127.0.0.1 or that the host has a firewall (either local system or NeCTAR security group e.g.) to block shiny server ports to outside sources"
- [readme] readme config note: "* Need to write your own config.py! See the sample config.py included"
