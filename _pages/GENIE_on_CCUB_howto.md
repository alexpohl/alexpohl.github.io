---
layout: smallfont_page
permalink: /GENIE_on_CCUB_howto/
title: Installing GENIE on the CCUB cluster
description: Changing paths to run the model on the work directory
nav: false
---

Clone cgenie.muffin from Github onto your work dir.
```
git clone https://github.com/derpycode/cgenie.muffin.git
```

Then, a few files need to be changed in `cgenie.muffin/genie-main`:

# File user.mak
```
GENIE_ROOT        = /work/crct/al1966po/cgenie.muffin
OUT_DIR           = /work/crct/al1966po/cgenie.output
...
# === NetCDF library ===
NETCDF_DIR=/user1/crct/al1966po/NetCDF_for_GENIE
```

# Files runmuffin.sh and runmuffin_nocleanall.sh
```
HOMEDIR=/work/crct/al1966po
```

# File user.sh
```
CODEDIR=/work/crct/al1966po/cgenie.muffin
OUTROOT=/work/crct/al1966po/cgenie_output
ARCHIVEDIR=/work/crct/al1966po/cgenie_archive
LOGDIR=/work/crct/al1966po/cgenie_log
```

# File makefile.arc
```
(line 575)
NETCDF= $(LIB_SEARCH_FLAG)$(PATH_QUOTE)$(NETCDF_DIR)/lib$(PATH_QUOTE) $(LIB_FLAG)$(NETCDF_NAMEF) $(LIB_FLAG)$(NETCDF_NAME)
```

You will have to get the NetCDF library found here: `/user1/crct/al1966po/NetCDF_for_GENIE`.

Beyond this, all info needed can be found [[here](https://github.com/derpycode/muffindoc){:target="_blank"}] in the official documentation.
