---
layout: smallfont_page
permalink: /GENIE_summer_school_2025/
title: cGENIE summer school (UBE graduate programme) 2025
description: Instructions to connect to the CCUB cluster and get GENIE running
nav: false
---
# Connecting to the cluster of Univ. Bourgogne Europe with NoMachine

- Open the `NoMachine` app.
- Use your CCUB login and password to connect.
- You are ready to open a terminal window to work (Menu > Emulateur de terminal); would you be given a choice, choose the terminal called Rxfce, which is black by default.
- When needed, you can use the local `FileZilla` application to transfer files between the local computer and the remote cluster using a graphical interface.

__Remark:__ it is also possible to just connect without graphical interface through ssh: 
```
ssh -Y -C -o StrictHostKeyChecking=no CCUBlogin@krenek2002.u-bourgogne.fr
```
… and to transfer files between the cluster and your local computer through scp.

# Downloading and installing cGENIE

## Go to the workdir (this is where the model is going to run)

`cd /work/crct/CCUBlogin` (with CCUBlogin, your CCUB login)

## Downloading model source code

`git clone https://github.com/derpycode/cgenie.muffin.git`


