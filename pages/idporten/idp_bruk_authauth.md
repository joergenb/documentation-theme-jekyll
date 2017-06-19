---
layout: page
title: "Autentiseringsnær autorisasjon"
sidebar: idporten
permalink: idp_bruk_authauth.html

summary: "I forbindelse med en OpenID Connect-autentisering kan ID-portens OpenID Connect provider også autorisere en tjeneste til å opptre på vegne av innbyggeren opp mot et API tilbudt av en 3.dje-part."
---



## Om bruksområdet

Dette er den klassise Oauth2-flyten, der innbyggeren samtykker - enten eksplisitt eller implisitt - til at tjenesten kan bruke et API på vegne av seg selv. I ID-porten-sammenheng vil vanligvis samtykket være implisitt, siden det er autentiseringshandlingen som i seg selv tolkes som det informerte samtykket (“Ved å logge inn i tjenesten godtar du at vi henter opplysninger om deg fra Vegveseent”).

Samtykket, eller autorisasjonen, blir av ID-porten utlevert som et access_token. Tjenesten bruker så dette access_tokenet når den skal aksessere APIet.

<div class="mermaid">
sequenceDiagram

Bruker->>Klient: Ønsker å bruke tjeneste
Klient->>AS: Kan eg få? (authorize/)
AS->>Bruker: Er det greitt?  (innlogging +evt. samtykke)
Bruker->>AS: Jada
AS->>Klient: Returnerer tokens (Access_token + evt id_token)
Klient->>RS: Bruk API (med access_token)
RS->>AS:  Validere token (access_token)
AS->>RS: OK
RS->>Klient: Resultat av API-operasjon
</div>

Merk at Oauth2 kun omhandler selve _autorisasjon_, men at autorisasjons-serveren normalt vil måtte be brukeren _autentisere_ seg for å vite hvem brukeren er.

### om samtykker

samtykke kan vere eksplisitt (bruker får presentert en side med "ønsker du at Instagram skal kunne poste på Facebook-veggen din ?"  eller implisitt (ikke noe egen siden, men annen informasjon, for eksempel i tilknytting til tjeenste "ved å logge inn i denne tjenesten godtar du at vi henter opplysninger hos X"


## Overordnet systemlandskap

I offentlig sektor har vi mange aktører som bruker Oauth2/OpenID Connect.   Figuren under viser en overorndet arkitektur.

<div class="mermaid">

graph LR

 subgraph autorisasjonslag
  dataporten
  altinn 
  idporten-oidc
  ehelse-as
 end

 subgraph API-lag
  mittapi["Mitt API"]
  dittapi["Annet API"]

  mittapi-. kontrollerer tilgang .- dataporten
  dittapi-. kontrollerer tilgang .- altinn
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


### ulike typer klienter

| Klient | Beskrivelse | Anbefalinger |
| --- | --- | --- |
| Nett-tjeneste | Tradisjonell nett-tjeneste med applikasjonsserver i sikret miljø. <br/> AS kan stole på at klient-hemmelighet / virksomhetssertifikat ikke kommer på avveie. <br/><br/>= confidential klient ihht. oauth2. | Bruk authorization code flow. <br/>PKCE ikke nødvendig.<br/>Tilstrekkelig med self-contained tokens (som oftest)<br/>Bruk refresh_token dersom token skal vare "lenge" |
| SPA / Browser-tjeneste | Tjenesten kjører i brukers browser, typisk single-page javascript applikasjon.<br/>= public client ihtt oauth2.<br/> AS kan ikke stole på at tjenesten kan holde på en hemmelighet / virksomhetssertifikat, eller beskytte tokens over lengre tid. <br/> SPAen trenger strengt tatt ikke motta id-token selv, dvs. kun behov for autorisasjon for å aksessere APIer |For API’er: bruk by-reference access_tokens og finn identitet fra /tokeninfo hos AS.<br/>Bruk: implicit flow <br/> Ikke bruk refresh_token, gi ut kort-levde access_token, og evt. bruk SSO i AS/ OIDC provider for at brukeren skal slippe å logge inn igjen. |
| Mobil-app | = oauth2 native app | Bruk: code flow + PKCE (for å beskytte seg mot «slemme» apper)<br/> Kan motta og persistere refresh_tokens ?<br/> Vudere dynamic client registreration, med instans-spesifikk client_id. (evt. med client-secret) |





