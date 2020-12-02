---
title: About the preCICE project
permalink: about.html
keywords: about, project leaders, timeline, impressum, legal
summary:
hide_sidebar: true
toc: false
editme: false
---

## Development

preCICE is developed at the [Chair of Scientific Computing](http://www5.in.tum.de/wiki/index.php/Home) at the Technical University of Munich and at the [Institute for Parallel and Distributed Systems](http://www.ipvs.uni-stuttgart.de/) at the University of Stuttgart.

For the full list of contributors to preCICE please see our [community page](community-contributors).

## Scientific project leaders

<br>
{% assign dev_leads = site.data.developer.leads | sort: "name" %}
<div class="row">
<div class="col-md-8 col-md-offset-2">
<ul class="devlist">
  {% for p in dev_leads %}
  <li{% if forloop.first %} class="devlist-first"{% endif %}>
    <div class="devlist-img">
      {% if p.img %}
      <img src="images/developer/{{ p.img }}.jpg" alt="Portait">
      {% endif %}
    </div>
    <div class="devlist-left">
      <p>
        <strong>{{ p.fullname }}</strong><br/>
        {{ p.institution }}
      </p>
    </div>
    <ul class="devlist-right">
      {% if p.url %}<li><a href="{{ p.url }}" alt="See the institutional website" class="notExternal"><i class="fas fa-university"></i></a></li>{% endif %}
      {% if p.github %}<li><a href="https://github.com/{{ p.github }}" alt="See the Github profile" class="notExternal"><i class="fab fa-github"></i></a></li>{% endif %}
    </ul>
  </li>
  {% endfor %}
</ul>
</div>
</div>

The conceptual ideas of preCICE are not completely new, preCICE is an advancement of FSI*ce, which has been developed by Dr. Markus Brenk.

## Funding

<div class="row vertical-align">
<div class="col-md-2 col-md-offset-2 col-xs-4">
  <a class="notExternal" target="_blank" href="http://www.sppexa.de/"><img class="img-responsive" src="images/funding/sppexa.jpg" alt="SPPEXA"></a>
</div>
<div class="col-md-2 col-md-offset-1 col-xs-4">
  <a class="notExternal" target="_blank" href="http://gepris.dfg.de/gepris/projekt/391150578"><img class="img-responsive" src="images/funding/dfg.jpg" alt="preDOM"></a>
</div>
<div class="col-md-2 col-md-offset-1 col-xs-4">
  <a class="notExternal" target="_blank" href="https://www.bmwi.de/"><img class="img-responsive" src="images/funding/bmwi.png" alt="BMWi"></a>
</div>
</div>
<div class="row vertical-align">
<div class="col-md-2 col-md-offset-3 col-xs-4  col-xs-offset-1">
  <a class="notExternal" target="_blank" href="http://postdoc.eurotech-universities.eu/"><img class="img-responsive" src="images/funding/eurotech.jpeg" alt="EuroTech"></a>
</div>
<div class="col-md-4 col-md-offset-1 col-xs-6 col-xs-offset-1">
  <img class="img-responsive center-block" src="images/funding/eu.png" alt="European Union" style="width: 35%; margin: auto;">
  <br>
  <p style="font-size:.7em;">This project has received funding from the European Union’s Horizon 2020 research and innovation programme under the Marie Skłodowska-Curie grant agreement No 754462</p>
</div>
</div>

## Impressum

This website is maintained by


Benjamin Uekermann  
Eindhoven University of Technology  
Department of Mechanical Engineering  
Energy Technology  
P.O. Box 513 | 5600MB EINDHOVEN | The Netherlands


![preCICE doughnuts](images/doughnuts.jpg)

preCICE is tasty because it is made with love by its contributors.