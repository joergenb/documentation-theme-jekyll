---
layout: page
title: Systemlandskap
folder: oauth2

---

Følgende moment har vært viktige ved utarbeidelse av disse veilederne:

* Følge standardene i så stor grad som mulig
* Bred støtte i etablerte rammeverk og produkter
* Støtte distributert arkitektur med automome sektor-spesifikke løsninger
* Understøtte bruk av  felleskomponenter
* Sektor-spesifikke løsninger må samvirke 


Overordnet systemlandskap
----


<div class="mermaid">
graph LR

 subgraph autorisasjonslag
  altinn 
  dataporten
  idporten-oidc
  ehelse-as
 end

 subgraph API-lag
  mittapi["Mitt API"]
  dittapi["Annet API"]

  mittapi-. kontrollerer tilgang .- altinn
  dittapi-. kontrollerer tilgang .- dataporten
 end

 subgraph Klienter 
  subgraph Tradisjonell netttjeneste
    browser
    applikasjonsserver
    browser-->applikasjonsserver
  end
  SPA
  native["Mobil-app"]

  applikasjonsserver-->mittapi
  applikasjonsserver-->dittapi
  applikasjonsserver-- Ber om tilgang --- dataporten
  applikasjonsserver-- Ber om tilgang --- altinn

  SPA-->mittapi
  native-->mittapi
 end
 

</div>
