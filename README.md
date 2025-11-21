-- if not getgenv().directory then
--     game.Players.LocalPlayer:Kick("Script directory was not found!\nNOTE: THE SERVER MAY BE DOWN \nplease dm 1catlol if you have entered yours correctly")
--     wait(3)
--     game.Players.LocalPlayer:Destroy()
--     return LPH_CRASH()
-- end

-- hookfunction security
    local exe_name, exe_version = identifyexecutor()
    local function home999() end
    local function home888() end

    if exe_name ~= "Wave Windows" then
        hookfunction(home888, home999)
        if isfunctionhooked(home888) == false then
            game.Players.LocalPlayer:Destroy()
            return LPH_CRASH()
        end
    end 
    
    local function check_env(env)
        for _, func in env do
            if type(func) ~= "function" then
                continue
            end

            local functionhook = isfunctionhooked(func)

            if functionhook then
                game.Players.LocalPlayer:Destroy()
                return LPH_CRASH()
            end
        end
    end

    check_env( getgenv() )
    check_env( getrenv() )
--

local Lua_Fetch_Connections = getconnections
local Lua_Fetch_Upvalues = getupvalues
local Lua_Hook = hookfunction 
local Lua_Hook_Method = hookmetamethod
local Lua_Unhook = restorefunction
local Lua_Replace_Function = replaceclosure
local Lua_Set_Upvalue = setupvalue
local Lua_Clone_Function = clonefunction

local Game_RunService = game:GetService("RunService")
local Game_LogService = game:GetService("LogService")
local Game_LogService_MessageOut = Game_LogService.MessageOut

local String_Lower = string.lower
local Table_Find = table.find
local Get_Type = type

local Current_Connections = {};
local Hooked_Connections = {};

local function Test_Table(Table, Return_Type)
    for TABLE_INDEX, TABLE_VALUE in Table do
        if type(TABLE_VALUE) == String_Lower(Return_Type) then
            return TABLE_VALUE, TABLE_INDEX
        end

        continue
    end
end

local function Print_Table(Table)
    table.foreach(Table, print)
end

if getgenv().DEBUG then
    print("[auth.injected.live] Waiting...")
end

local good_check = 0

function auth_heart()
    -- local avalible = pcall(function() return loadstring(game:HttpGet("https://auth.injected.live/" .. directory))() end)
    
    -- if (not avalible or not game:HttpGet("https://auth.injected.live/" .. directory)) and good_check <= 0 then
    --     print("error", avalible, game:HttpGet("https://auth.injected.live/" .. directory))
    --     game.Players.LocalPlayer:Destroy()
    --     return LPH_CRASH()
    -- end

    return true , true
end

function Lua_Common_Intercept(old, ...)
    print(...)
    return old(...)
end

function XVNP_L(CONNECTION)
    local s, e = pcall(function()
        local OPENAC_TABLE = Lua_Fetch_Upvalues(CONNECTION.Function)[9]
        local OPENAC_FUNCTION = OPENAC_TABLE[1]
        local IGNORED_INDEX = {3, 12, 1, 11, 15, 8, 20, 18, 22}

        --[[
            3(Getfenv), 1(create thread), 12(Some thread function errors btw), 11( buffer (BANS YOU) ), 8(BXOR), 14(WRAP), 15(YIELD), 22(JUNK), 20(Setfenv), 18(Idk for now)
        ]]


        Lua_Set_Upvalue(OPENAC_FUNCTION, 14, function(...)
            return function(...)
                local args = {...}

                if type(args[1]) == "table" and args[1][1] then
                    pcall(function()
                        if type(args[1][1]) == "userdata" then
                            args[1][1]:Disconnect()
                            args[1][2]:Disconnect()
                            args[1][3]:Disconnect()
                            args[1][4]:Disconnect()
                            --warn("[XVNP] DISCONNECTING CURRENT FUNCTIONS")
                        end

                        --Print_Table(args[1])
                    end)
                end 
            end
        end)

        Lua_Set_Upvalue(OPENAC_FUNCTION, 1, function(...)
            task.wait(200)
        end)

        hookfunction(OPENAC_FUNCTION, function(...)
            --warn("[XVNP DEBUG]", ...)
            return {}
        end)
    end)
end

local XVNP_LASTUPDATE = 0
local XVNP_UPDATEINTERVAL = 5

local XVNP_CONNECTIONSNIFFER;

XVNP_CONNECTIONSNIFFER = Game_RunService.RenderStepped:Connect(function()
    if #Lua_Fetch_Connections(Game_LogService_MessageOut) >= 2 then
        --print("[XVNP] !Emulator overflow!")
        XVNP_CONNECTIONSNIFFER:Disconnect()
    end

    if tick() - XVNP_LASTUPDATE >= XVNP_UPDATEINTERVAL then
        XVNP_LASTUPDATE = tick() 

        local OpenAc_Connections = Lua_Fetch_Connections(Game_LogService_MessageOut)

        for _, CONNECTION in OpenAc_Connections do
            if not table.find(Current_Connections, CONNECTION) then
                table.insert(Current_Connections, CONNECTION)
                table.insert(Hooked_Connections, CONNECTION)

                XVNP_L(CONNECTION)
                
            end
        end
    end
end)

local last_beat = 0
Game_RunService.RenderStepped:Connect(function()
    if last_beat + 1 < tick() then
        last_beat = tick() + 1 

        local what, are = auth_heart()

        if not are or not what then
            if good_check <= 0 then
                game.Players.LocalPlayer:Destroy()
                return LPH_CRASH()
            else
                good_check -=1
            end
        else
            good_check += 1
        end

    end
end)

if getgenv().DEBUG then
    print("[auth.injected.live] Started Emulation Thread")
end
