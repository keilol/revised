--====================================================--
-- SILENT AIM SYSTEM (MAIN EVENT / UpdateMousePosI2)
--====================================================--

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

--===============--
-- Silent Table
--===============--

local Silent = {
    Enabled = true,
    FOVEnabled = true,
    FOV = 120,
    HitChance = 100,
    PredictionEnabled = true,
    PredictionAmount = 0.12,
    ResolverEnabled = false,
    TargetPart = "Head",      -- Or "Nearest"
    Target = nil,
}

local SmartParts = {
    "Head", "UpperTorso", "LowerTorso", "HumanoidRootPart",
    "LeftUpperArm", "LeftLowerArm", "LeftHand",
    "RightUpperArm", "RightLowerArm", "RightHand",
    "LeftUpperLeg", "LeftLowerLeg", "LeftFoot",
    "RightUpperLeg", "RightLowerLeg", "RightFoot"
}

--========================--
-- FOV CIRCLE
--========================--

local FOV = Drawing.new("Circle")
FOV.Color = Color3.fromRGB(255,0,0)
FOV.Thickness = 2
FOV.Filled = false
FOV.Radius = Silent.FOV
FOV.Visible = true

--========================--
-- Smart Nearest Part
--========================--

local function GetNearestPart(char)
    local mouse = UIS:GetMouseLocation()
    local best, bestDist = nil, math.huge

    for _, name in ipairs(SmartParts) do
        local part = char:FindFirstChild(name)
        if part then
            local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X,pos.Y) - mouse).Magnitude
                if dist < bestDist then
                    bestDist = dist
                    best = part
                end
            end
        end
    end

    return best
end

--========================--
-- Find Silent Aim Target
--========================--

local function GetClosest()
    local mouse = UIS:GetMouseLocation()
    local closest, shortest = nil, math.huge

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Humanoid") then
            local hum = plr.Character.Humanoid
            if hum.Health > 0 then

                local part =
                    (Silent.TargetPart == "Nearest" and GetNearestPart(plr.Character))
                    or plr.Character:FindFirstChild(Silent.TargetPart)

                if part then
                    local pos, vis = Camera:WorldToViewportPoint(part.Position)
                    if vis then
                        local dist = (Vector2.new(pos.X,pos.Y)-mouse).Magnitude

                        if (not Silent.FOVEnabled) or dist <= Silent.FOV then
                            if dist < shortest then
                                shortest = dist
                                closest = plr
                            end
                        end
                    end
                end
            end
        end
    end

    return closest
end

-- Update target each frame
RunService.RenderStepped:Connect(function()
    FOV.Position = UIS:GetMouseLocation()
    FOV.Radius = Silent.FOV
    FOV.Visible = Silent.Enabled and Silent.FOVEnabled

    Silent.Target = GetClosest()
end)

--========================================--
-- SILENT AIM HOOK (MainEvent / UpdateMousePosI2)
--========================================--

local MainEvent = ReplicatedStorage:FindFirstChild("MainEvent")
if not MainEvent then
    warn("Silent Aim: MainEvent not found!")
    return
end

local mt = getrawmetatable(game)
setreadonly(mt, false)

local old = mt.__namecall

mt.__namecall = newcclosure(function(self, ...)
    local args = {...}
    local method = getnamecallmethod()

    -- Only modify THIS specific call:
    if self == MainEvent
        and method == "FireServer"
        and args[1] == "UpdateMousePosI2"
        and Silent.Enabled
        and Silent.Target
        and Silent.Target.Character
    then
        local char = Silent.Target.Character

        -- Which part to aim at?
        local part =
            (Silent.TargetPart == "Nearest" and GetNearestPart(char))
            or char:FindFirstChild(Silent.TargetPart)

        if not part then
            return old(self, ...)
        end

        -- HitChance check
        if math.random(1,100) > Silent.HitChance then
            return old(self, ...)
        end

        -- Apply prediction
        local pos = part.Position
        local HRP = char:FindFirstChild("HumanoidRootPart")

        if HRP then
            if Silent.ResolverEnabled then
                pos = pos + HRP.Velocity * (Silent.PredictionAmount + 0.05)
            elseif Silent.PredictionEnabled then
                pos = pos + HRP.Velocity * Silent.PredictionAmount
            end
        end

        -- Replace argument 2 with silently-aimed position
        args[2] = pos

        return old(self, unpack(args))
    end

    return old(self, ...)
end)

setreadonly(mt, true)

print("Silent Aim Loaded (MainEvent / UpdateMousePosI2)")
