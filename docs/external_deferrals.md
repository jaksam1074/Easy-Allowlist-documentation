## Disable default easy_allowlist deferrals

In case your framework uses deferrals itself, you can disable the default deferrals used by easy_allowlist. 

You have to add the following code in ```easy_allowlist/integrations/sv_integrations.lua```

```lua
Citizen.CreateThread(function() 
    exports["easy_allowlist"]:disableScriptEvent("playerConnecting")
end)
```

And you'll have to manually add the event in your framework script that use deferrals, by using the following event server side, replacing deferrals.done()

```lua
TriggerEvent("easy_allowlist:startDeferringPlayer", source, deferrals)
```

## Example for QB-Core

**Old code**
```lua
-- Path: qb-core/server/events.lua

local function OnPlayerConnecting(name, setKickReason, deferrals)
    local player = source
    local license
    local identifiers = GetPlayerIdentifiers(player)
    deferrals.defer()

    -- mandatory wait!
    Wait(0)

    deferrals.update(string.format('Hello %s. Validating Your Rockstar License', name))

    for _, v in pairs(identifiers) do
        if string.find(v, 'license') then
            license = v
            break
        end
    end

    -- mandatory wait!
    Wait(2500)

    deferrals.update(string.format('Hello %s. We are checking if you are banned.', name))

    local isBanned, Reason = QBCore.Functions.IsPlayerBanned(player)
    local isLicenseAlreadyInUse = QBCore.Functions.IsLicenseInUse(license)

    Wait(2500)

    deferrals.update(string.format('Welcome %s to {Server Name}.', name))

    if not license then
        deferrals.done('No Valid Rockstar License Found')
    elseif isBanned then
        deferrals.done(Reason)
    elseif isLicenseAlreadyInUse then
        deferrals.done('Duplicate Rockstar License Found')
    else
        deferrals.done()
        Wait(1000)
        TriggerEvent('connectqueue:playerConnect', name, setKickReason, deferrals)
    end
end
```

**New code**
```lua
-- Path: qb-core/server/events.lua

local function OnPlayerConnecting(name, setKickReason, deferrals)
    local player = source
    local license
    local identifiers = GetPlayerIdentifiers(player)
    deferrals.defer()

    -- mandatory wait!
    Wait(0)

    deferrals.update(string.format('Hello %s. Validating Your Rockstar License', name))

    for _, v in pairs(identifiers) do
        if string.find(v, 'license') then
            license = v
            break
        end
    end

    -- mandatory wait!
    Wait(2500)

    deferrals.update(string.format('Hello %s. We are checking if you are banned.', name))

    local isBanned, Reason = QBCore.Functions.IsPlayerBanned(player)
    local isLicenseAlreadyInUse = QBCore.Functions.IsLicenseInUse(license)

    Wait(2500)

    deferrals.update(string.format('Welcome %s to {Server Name}.', name))

    if not license then
        deferrals.done('No Valid Rockstar License Found')
    elseif isBanned then
        deferrals.done(Reason)
    elseif isLicenseAlreadyInUse then
        deferrals.done('Duplicate Rockstar License Found')
    else
        --[[
            -- REMOVES THE OLD deferrals.done() and the queue since easy_allowlist also provides you a queue system
            deferrals.done()
            Wait(1000)
            TriggerEvent('connectqueue:playerConnect', name, setKickReason, deferrals)
        ]]

        -- easy_allowlist system
        TriggerEvent("easy_allowlist:startDeferringPlayer", player, deferrals)
    end
end
```

## Disable default QB-Core queue
To disable the default QB-Core queue, you can delete the ```connectqueue``` script folder and remove the dependency in qb-core script

**Example Path: qb-core/fxmanifest.lua**
```lua
-- OLD CODE
dependencies {
	'oxmysql',
	'progressbar',
	'connectqueue'
}
```
```lua
-- NEW CODE
dependencies {
	'oxmysql',
	'progressbar'
}
```