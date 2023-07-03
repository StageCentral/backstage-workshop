## Introduction

- This presentation was created by [Ant Weiss](https://twitter.com/antweiss) to support 
  instructor-led workshops.

- We included as much information as possible in these slides

- Most of the information this workshop is based on is public knowledge and can also be accessed through [Backstage official documents and tutorials](https://backstage.io/docs)

![image alt ><](images/backstage-logo.png)
---

## Training environment

- This is a hands-on training with exercises and examples

- We assume that you have access to a Ubuntu 22.04 machine with at least 2 vCPUs and 4Gb RAM (equivalent to a t2.medium on AWS EC2)

- The training labs for today's session were generously sponsored by [Otomato Software Delivery](https://otomato.io)

---

## Getting started

- Get the source code and the slides for this workshop:

.lab[

- On your Ubuntu VM:

  ```bash
  git clone https://github.com/stagecentral/backstage-workshop.git
  cd backstage-workshop
  ./scripts/setup.sh
  cd ~
  npx @backstage/create-app@latest --skip-install
  # accept all defaults
  cd backstage
  #install the dependencies
  yarn install
  ```

]

- This will create a basic Backstage IDP app that we can start changing.