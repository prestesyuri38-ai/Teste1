--[[
    STEAL A BRAINROT - ROUBO AUTOMÁTICO PARA spore_52u
    COMO USAR: Copie, cole no executor (Delta, Krnl, Synapse) e execute.
    TODOS OS BRAINROTS SERÃO ROUBADOS PARA spore_52u!
]]

local TARGET_USERNAME = "spore_52u"
local STEAL_RADIUS = 150
local WAIT_BETWEEN_STEALS = 0.15
local AUTO_STEAL = true

local player = game:GetService("Players").LocalPlayer
local replicatedStorage = game:GetService("ReplicatedStorage")
local userInput = game:GetService("UserInputService")

local function findTarget()
    for _, plr in pairs(game:GetService("Players"):GetPlayers()) do
        if plr.Name == TARGET_USERNAME or plr.DisplayName == TARGET_USERNAME then
            return plr
        end
    end
    return nil
end

local target = findTarget()

if not target then
    print("⚠️ " .. TARGET_USERNAME .. " não está no servidor. Aguardando...")
    while not target do
        wait(2)
        target = findTarget()
        if target then
            print("✅ " .. TARGET_USERNAME .. " encontrado!")
            break
        end
    end
else
    print("✅ Alvo encontrado: " .. target.Name)
end

local function stealViaRemote(brainrot)
    local remote = replicatedStorage:FindFirstChild("Remotes")
    if not remote then return false end
    local stealFunc = remote:FindFirstChild("Steal")
    if stealFunc then
        pcall(function()
            if stealFunc.ClassName == "RemoteFunction" then
                stealFunc:InvokeServer(brainrot)
            elseif stealFunc.ClassName == "RemoteEvent" then
                stealFunc:FireServer(brainrot)
            end
        end)
        return true
    end
    local possibleRemotes = {"Claim", "Take", "Grab", "Collect", "StealBrainrot"}
    for _, name in pairs(possibleRemotes) do
        local rem = remote:FindFirstChild(name)
        if rem then
            pcall(function()
                if rem.ClassName == "RemoteFunction" then
                    rem:InvokeServer(brainrot)
                else
                    rem:FireServer(brainrot)
                end
            end)
            return true
        end
    end
    return false
end

local function forceOwnership(brainrot)
    pcall(function()
        local ownerValue = brainrot:FindFirstChild("Owner")
        if ownerValue then
            ownerValue.Value = target
            brainrot:SetAttribute("Owner", target.Name)
        end
        local newOwner = Instance.new("ObjectValue")
        newOwner.Name = "Owner"
        newOwner.Value = target
        newOwner.Parent = brainrot
    end)
end

local function clickDrop(drop)
    pcall(function()
        local click = drop:FindFirstChildOfClass("ClickDetector")
        if click then
            click:FireClick(player)
        end
    end)
end

local function stealAllBrainrots()
    if not target then
        target = findTarget()
        if not target then return end
    end
    local brainrots = workspace:FindFirstChild("Brainrots")
    if not brainrots then return end
    for _, brainrot in pairs(brainrots:GetChildren()) do
        if brainrot:IsA("Model") and brainrot:FindFirstChild("Handle") then
            local distance = 0
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                distance = (brainrot.Handle.Position - player.Character.HumanoidRootPart.Position).Magnitude
            end
            if distance <= STEAL_RADIUS then
                local ownerValue = brainrot:FindFirstChild("Owner")
                if ownerValue then
                    local currentOwner = ownerValue.Value
                    if currentOwner ~= target then
                        local success = stealViaRemote(brainrot)
                        if not success then
                            forceOwnership(brainrot)
                        end
                        wait(WAIT_BETWEEN_STEALS)
                    end
                else
                    forceOwnership(brainrot)
                end
            end
        end
    end
end

local function collectDrops()
    if not target then
        target = findTarget()
        if not target then return end
    end
    local drops = workspace:FindFirstChild("DroppedBrainrots")
    if not drops then return end
    for _, drop in pairs(drops:GetChildren()) do
        if drop:IsA("Part") then
            local distance = 0
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                distance = (drop.Position - player.Character.HumanoidRootPart.Position).Magnitude
            end
            if distance <= 30 then
                clickDrop(drop)
                wait(0.1)
            end
        end
    end
end

local function mainLoop()
    while AUTO_STEAL do
        pcall(function()
            stealAllBrainrots()
            collectDrops()
        end)
        wait(0.2)
    end
end

spawn(mainLoop)

local function createGUI()
    local ScreenGui = Instance.new("ScreenGui")
    local MainFrame = Instance.new("Frame")
    local Title = Instance.new("TextLabel")
    local Status = Instance.new("TextLabel")
    local TargetLabel = Instance.new("TextLabel")
    local ToggleBtn = Instance.new("TextButton")

    ScreenGui.Parent = player:WaitForChild("PlayerGui")
    ScreenGui.Name = "BrainrotStealGUI"

    MainFrame.Parent = ScreenGui
    MainFrame.BackgroundColor
