### Foliant

[TOC]

I attempted two approaches: 

- Using docker, this doesn't work in all features compared to local. **Don't waste time using docker.**
- Using local is the best option, but  more validation required.

#### # Docker approach - don't use

Except for docker, you shouldn't need python3 on local.  

##### First time setup

```bash
# Assume docker, docker-compose, docker-machine, docker.app is already installed.
$ docker pull foliant/foliant

$ cd source # some working directory.
$ docker run --rm -it -v `pwd`:/usr/app/src -w /usr/app/src foliant/foliant init
# project name is Hello Foliant. A project folder will be created.

$ cd hello-foliant

# build site, a subfolder <project-name>.<timestamp>.mkdocs will be created.
$ docker-compose run --rm <project-name> make site
$ cd <project-name>.mkdocs
$ python3 -m http.server
# On browser go to localhost:8000
# Open another terminal window and go to the project folder.
```

##### Add preprocessors

Include one or more preprocessors (such as plant-uml) to validate them.

**Dockerfile**

Update the `Dockerfile` as follows:

```dockerfile
FROM foliant/foliant
# If you plan to bake PDFs, uncomment this line and comment the line above:
# FROM foliant/foliant:pandoc

RUN pip3 install foliantcontrib.admonitions
RUN pip3 install foliantcontrib.blockdiag
RUN pip3 install foliantcontrib.glossary
RUN pip3 install foliantcontrib.graphviz
RUN pip3 install foliantcontrib.plantuml
RUN pip3 install foliantcontrib.csvtables

COPY requirements.txt .
RUN pip3 install -r requirements.txt
```

**foliant.yml**

Update `Foliant.yml` as follows:

```yaml
title: Hello Foliant
chapters:
  - index.md
  - hello.md
preprocessors:
  - admonitions
  - blockdiag
  - csvtables
  - glossary
  - graphviz
  - plantuml:
      parse_raw: true
```

**hello.md**

Create a markdown inside the `src` child folder.

```markdown
# Hello Again

This is regular text generated from regular Markdown.
Foliant doesn't force any *special* Markdown flavor.

## Heading 2
some desc

### Heading 3
some more desc

## Preprocessor examples

### blockdiag

<blockdiag caption="High-quality svg diagram" format="svg">
  blockdiag {
    A -> B -> C -> D;
    A -> E -> F -> G;
  }
</blockdiag>

<seqdiag caption="High-quality svg diagram" format="svg">
  seqdiag {
    browser  -> webserver [label = "GET /index.html"];
    browser <-- webserver;
    browser  -> webserver [label = "POST /blog/comment"];
                webserver  -> database [label = "INSERT comment"];
                webserver <-- database;
    browser <-- webserver;
  }
</seqdiag>

### plantuml

<plantuml caption="Sample diagram from the official site" format="eps">
    @startuml
        a->b:Request
        b-->a:Response
    @enduml
</plantuml>

@startuml
  a->b:Request without plantuml preprocessor tag
  b-->a:Response
@enduml

### csvtables

<csvtable>
    Header 1;Header 2;Header 3;Header 4;Header 5
    Datum 1;Datum 2;Datum 3;Datum 4;Datum 5
    Datum 6;Datum 7;Datum 8;Datum 9;Datum 10
</csvtable>

### admonitions

!!! warning "optional admonition title"
    Admonition text.
    May be several paragraphs.

!!! type "optional explicit title within double quotes"
    Any number of other indented markdown elements.

    This is the second paragraph.

!!! note
    You should note that the title will be automatically capitalized.

!!! danger "Don't try this at home"
    You should note that the title will be automatically capitalized.

!!! important ""
    This is an admonition box without a title.

### graphviz

<graphviz>
    a -> b
    b -> c
</graphviz>

## Glossary

:   Term 1
:   Term 4
:   Term 2

```

**term_definitions.md**

Create this markdown _in the top-level project root._ 

```markdown
## Glossary

Term 1
:   Definition 1

Term 2
:   Definition 2

Term 3
:   Definition 3

Term 4
:   Definition 4
        { some code, part of Definition 4 }
    Third paragraph of definition 4.

Term 5
:   Definition 5

AISP
:   Account Information Service Provider provides account information services as an online service to provide consolidated information on one or more payment accounts held by a payment service user with one or more payment service provider(s).

PISP
:   Payment Initiated Service Provider provides an online service to initiate a payment order at the request of the payment service user with respect to a payment account held at another payment service provider.

ASPSP
:   Account Servicing Payment Service Provider is a Bank or Financial Institution in the context of Open Banking Ecosystem that publish Read/Write APIs to permit, with customer consent, payments initiated by third party providers and/or make their customers’ account transaction data available to third party providers via their API end points.

CMA
:   Competition And Markets Authority is a non-ministerial government department in the United Kingdom, responsible for strengthening business competition and preventing and reducing anti-competitive activities.

CMA 9
:   The nine largest banks and building societies in Great Britain and Northern Ireland, based on the volume of personal and business current accounts.

    1.AIB Group (UK):[ First Trust Bank, Allied Irish Bank] 2. Barclays Bank  3. HSBC Group: [First Direct, Mark & Spencer, HSBC-UK] 4. Lloyds Banking Group [ Halifax, Lloyds Bank, Bank of Scotland] 5. Nationwide Building Society 6. Danske Bank 7. The Royal Bank of Scotland Group: [ Royal Bank of Scotland, Ulster Bank Ltd, National Westminister] 8. Santander UK.

Competent Authority
:   A Competent Authority, in the context of the Open Banking Ecosystem, is a governmental body or regulatory or supervisory authority having responsibility for the regulation or supervision of the subject matter of Participants.

OBD
:   The Open Banking Directory provides a “whitelist” of participants able to operate in the Open Banking Ecosystem, as required by the CMA Order.

    The Read/Write Directory also provides identity and access management services to provide identity information in order to participate in payment initiation and account information transactions through APIs.

OBDS
:   The Open Banking Directory Sandbox is a test instance of the Directory. The Directory Sandbox may be used to support testing applications with test API endpoints and testing integration with the Open Banking Directory.

FCA
:   Financial Conduct Authority is the conduct regulator for 56,000 financial services firms and financial markets in the UK and the prudential regulator for over 18,000 of those firms.

OBIE
:   Open Banking Implementation Entity is the delivery organisation working with the CMA9 and other stakeholders to define and develop the required APIs, security and messaging standards that underpin Open Banking. Otherwise known as Open Banking Limited.

PSD2
:   Revised Payment Services Directive 2015/2366, as amended or updated from time to time and including the associated Regulatory Technical Standards developed by the EBA and agreed by the European Commission and as implemented by the PSR and including any formal guidance issued by a Competent Authority.

PSP
:   Payment Service Provider is an entity which carries out regulated payment services, including AISPs, PISPs, CBPIIs and ASPSPs.

TPP
:   Third Party Provider is an organisation or natural person that use APIs developed to Standards to access customer’s accounts, in order to provide account information services and/or to initiate payments.

    Third Party Providers are either/both Payment Initiation Service Providers (PISPs) and/or Account Information Service Providers (AISPs).

SCA
:   Strong Customer Authentication is form of 2-factor authentication including password authentication <br>Example: Password authentication and something only the user possesses [for example, a particular cell phone and number]) and inherence (something the user is [or has, for example, a finger print or iris pattern]) that are independent, [so] the breach of one does not compromise the others, and is designed in such a way as to protect the confidentiality of the authentication data.

PSU
:   Payment Services User is a natural or legal person making use of a payment service as a payee, payer or both.

eiDAS
:   Electronic Identification, Authentication & Trust Service Regulation. The eIDAS regulation sets the standards across the European Union (EU) required for Trust Service Providers (TSPs) and the provision of trust services through technical mechanisms, such as Digital Certificates and Cryptographic Signatures

QTSP
:   Qualified Trust Service Provider. Qualified Trust Service Provider as permitted by MSSBs such as NCA (National Conduct Authority) & FCA ( Financial Conduct Authority) to issue Digital Certificates that are recognised across the EU.

    Trust Services Providers (TSPs) can freely passport their services within the EU, without the need for additional confirmation. These providers issue  QWAC and QSEALC certificates and are kin to Certificate Authority (CA's) in US.

QWAC
:   Qualified certificate for website authentication. Qualified Digital Certificate under the trust services defined in the eIDAS regulation and are used for website authentication, so that ASPSP and TPP can be certain of each other’s identity.

QSEALC
:   Qualified certificate for electronic seal. Qualified Digital Certificate under the trust services defined in the eIDAS regulation and used for identity verification, to protect transaction information from potential attacks during or after a communication.

```

##### Rebuild the site

From the project root, do the following:

```bash
$ docker-compose build

# Generates the site, and refresh the browser.
$ docker-compose run --rm <project-name> make site

# Generates the mkdocs source that can later generate the site.
# This is for experimentation only. 
$ docker-compose run --rm <project-name> make mkdocs
```

##### Observation

- Plantuml, Graphviz, Admonitions don't work. I have raised a issue on github.
- Blockdiag, Csvtables, Glossary work.

#### # Local approach

This requires python3 on local.  Instructions here are different from docker approach, so don't skip them.

```bash
$ python3 -m pip install foliant foliantcontrib.init
$ foliant init # Project name is Hello Foliant
$ cd hello-foliant
$ pip3 install -r requirements.txt
$ foliant make mkdocs # experimental to see mkdocs code generation
$ foliant make site
$ cd <project-name>..mkdocs
$ python3 -m http.server
# On browser go to localhost:8000
# Open another terminal window and go to the project folder. 
```
**term_definitions.md**

Add  `term_definitions.md` as described in the docker approach

**Update `src/` folder**

Extract the supplied zip into the `src` folder, and should look like:

![image-20191013091237591](/Users/mdeliw/Documents/Info/image-20191013091237591.png)

**Install the preprocessors**

```bash
$ pip3 install foliantcontrib.admonitions
$ pip3 install foliantcontrib.blockdiag
$ pip3 install foliantcontrib.glossary
$ pip3 install foliantcontrib.graphviz
$ pip3 install foliantcontrib.includes # to remove some warnings
$ pip3 install foliantcontrib.plantuml
$ pip3 install foliantcontrib.csvtables
```

**foliant.yml**

This is different from the docker version. 

```yaml
title: Hello Foliant

chapters:
  - index.md
escape_code: true
preprocessors:
  - admonitions
  - blockdiag
  - csvtables
  - glossary
  - graphviz
  - includes
  - plantuml:
      parse_raw: true
backend_config:
  mkdocs:
    use_title: true
    use_chapters: true
    use_headings: true
    default_subsection_title: Expand
    mkdocs.yml:
      site_url: http://dp.aexp.com
      site_author: Digital Payments
      markdown_extensions:
        - admonition
        - codehilite:
            linenums: true
        - footnotes
        - meta
        - pymdownx.arithmatex
        - pymdownx.betterem:
            smart_enable: all
        - pymdownx.caret
        - pymdownx.critic
        - pymdownx.details
        - pymdownx.emoji:
            emoji_generator: !!python/name:pymdownx.emoji.to_svg
        - pymdownx.inlinehilite
        - pymdownx.magiclink
        - pymdownx.mark
        - pymdownx.smartsymbols
        - pymdownx.superfences
        - pymdownx.tasklist:
            custom_checkbox: true
        - pymdownx.tilde
        - toc:
            permalink: true
      nav:
        - Home: index.md
        - Preprocessors: example/hello.md
        - P2P:
          - Overview: p2p/p2p.md
        - Open Banking:
          - Overview: pisp/pisp.md
        - DevOps:
          - Best practice: devops/best-practice.md
          - Support P2P: devops/p2p-ops.md
          - Support PISP: devops/pisp-ops.md
          - Support SRC: devops/src-ops.md
#      theme:
#        name: readthedocs
#        name: material
#        name: rtd-dropdown

```

**Build site without specifying a Mkdocs theme**

```bash
$ foliant make site
$ foliant make mkdocs # optional
# Refresh browser
```

**Build site with custom themes**

Foliant includes mkdocs' default themes. Additional [custom themes](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes) have to be manually installed. 

Next, install one or more custom themes:

```bash
# install custom themes
$ pip3 install mkdocs-rtd-dropdown
$ pip3 install mkdocs-material
```

Update foliant.yml to uncomment the `theme` section and try one of the custom themes. The theme `readthedocs` comes default with foliant, so we didn't have to explicitly install it. Try the `material` theme.

```yaml
      theme:
#        name: readthedocs
        name: material
#        name: rtd-dropdown
```

**Build site with a custom Mkdocs theme**

```bash
$ foliant make site
$ foliant make mkdocs # optional
# Refresh browser
```

##### Observation

- Everything except Graphviz works. 
- In `foliant.yml` the `chapters` section is mandatory and unfortunately maps to a deprecated `pages` feature in MkDocs. MkDocs now uses a `nav` section in which the index.md by default is the first item in the global navigation bar. To get around this problem, include index.md in the `chapters` and nothing else, and rely on mkdocs configuration to drive the content.
- Good reference for mkdocs customization is in the [material](https://github.com/squidfunk/mkdocs-material/blob/master/mkdocs.yml) theme.
- How to achieve further JS customization will need more work, as we have to see if mkdocs configuration allows us that flexibility.
- Hexo's document tag and reading time feature are not available here, but if needed we can work on it.



END OF DOCUMENT