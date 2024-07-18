# rv-path-converter

A simple [RV](https://help.autodesk.com/view/SGSUB/ENU/?guid=SG_RV_rv_osrv_html)/[OpenRV](https://github.com/AcademySoftwareFoundation/OpenRV) package to swap paths of incoming source media.

> #### Note
>
> This is a work in progress and may or may not work.
>
> You should probably use [environment variables](https://aswf-openrv.readthedocs.io/en/latest/rv-manuals/rv-user-manual/rv-user-manual-chapter-k.html) instead.

### Path conversion with Flow Production Tracking integration

This package doesn't work in conjunction with the `Flow Production Tracking` integration package, which annnoying checks if the path is empty on the disk via `commands.existingFilesInSequence` _before_ the `incoming-source-path` event is even triggered:

##### shotgrid_fields.mu

```mu
function: mediaTypePathEmpty (bool; string mediaType, StringMap info)
{
    let name = "mt_" + mediaType,
        ret = true;

    string path = info.find (name, true);

    if (path neq nil && path != "")
    {
        if (mediaType == "Streaming")
        {
            return false;
        }

        path = extractLocalPathValue(path);

        //
        //  existingFiles doesn't know about %vV.
        //
        if (regex("%v|%V").match(path)) ret = false;
        else
        {
            ret = (commands.existingFilesInSequence(path).size() == 0);
        }
    }
    return ret;
```

RV Path [environment variables](https://aswf-openrv.readthedocs.io/en/latest/rv-manuals/rv-user-manual/rv-user-manual-chapter-k.html) do work thanks to this code that executes during `_computeInternalPre`

```mu
            if (mediaPath neq nil)
            {
                data.add (key, commands.undoPathSwapVars(mediaPath));
            }
```

`commands.undoPathSwapVars` is [undocumented](https://github.com/AcademySoftwareFoundation/OpenRV/blob/ad0e4d25cbdbc3e9fec98ac46d30d7a3daadd496/src/lib/app/mu_rvui/commands.mud#L981) but does seem to swap RV_OS_PATHs (despite having `undo` in the name)

### shotgrid_fields_config_custom.mu files

There is little documentation about this, but apparently you could override `fieldsCompute` in a [custom shotgrid_fields_config_custom.mu config file](https://community.shotgridsoftware.com/t/custom-fields-media-representations-via-shotgrid-fields-config-custom-mu/18843) and do path substitution there. Ultimately I gave up trying to figure out regex in `mu` and just used environment variables.

<br />
<br />

---

This package is based off code by Autodesk, Inc., licensed under `Apache-2.0`.
