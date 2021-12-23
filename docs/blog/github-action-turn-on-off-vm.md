---
author: Roel Fauconnier
author_gh_user: roel4ez
read_time: 5min
publish_date: 23-dec-2021
tags:
    - github-action
    - azure
title: Github Action for managing Azure VM
---

## Background

In one of the [projects](https://github.com/Azure/iotedge-lorawan-starterkit)
I'm involved in, there was the need to let a GitHub Action automatically turn on
or off an Azure VM in the pipeline. The VM is required for running the full
end-to-end test suite, but we don't want to have it running the whole time since
it incurs a considerate costs if it is just idling away.

## Solution

To enable this functionality, I created a reusable GitHub Action that can be
integrated in any other pipeline: [Power ON/OFF or DEALLOCATE a specific Azure VM](https://github.com/marketplace/actions/power-on-off-or-deallocate-a-specific-azure-vm)

It is very easy to use, just add step to any job in your Action:

```yaml
steps:
    - name: Power Azure VM
    uses: roel4ez/action-power-on-off-azure-vm@v1.0.1
    with:
        AZURE_VM_NAME: <VM_NAME>
        AZURE_RG_NAME: <RESOURCE_GROUP_NAME>
        POWER_SWITCH: <ON/OFFDEALLOCATE>
        AZURE_SP_CLIENTID: ${{ secrets.AZURE_SP_CLIENTID }}
        AZURE_SP_SECRET: ${{ secrets.AZURE_SP_SECRET }}
        AZURE_TENANTID: ${{ secrets.AZURE_TENANTID }}
```

For more info, refer to the [README](https://github.com/marketplace/actions/power-on-off-or-deallocate-a-specific-azure-vm) or have a look at the [code](https://github.com/roel4ez/action-power-on-off-azure-vm) yourself.
Let me know what you think!
