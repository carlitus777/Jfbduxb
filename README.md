
-- ROOM 13 HUB | DELTA

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local espOn = false
local highlightOn = false
local noclipOn = false
local speedOn = false

local normalSpeed = 16
local boostedSpeed = 35
local originalCollision = {}

local gui = Instance.new("ScreenGui")
gui.Name = "Room13Hub"
gui.ResetOnSpawn = false
gui.Parent = game:GetService("CoreGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.fromOffset(220,270)
frame.Position = UDim2.new(0.5,-110,0.4,0)
frame.BackgroundColor3 = Color3.fromRGB(18,18,18)
frame.Active = true
frame.Draggable = true

Instance.new("UICorner",frame).CornerRadius = UDim.new(0,10)

local title = Instance.new("TextLabel",frame)
title.Size = UDim2.new(1,0,0,35)
title.BackgroundColor3 = Color3.fromRGB(150,0,0)
title.Text = "ROOM 13 HUB"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 16

local function makeButton(text,y)
    local b = Instance.new("TextButton",frame)
    b.Position = UDim2.fromOffset(10,y)
    b.Size = UDim2.new(1,-20,0,35)
    b.BackgroundColor3 = Color3.fromRGB(65,20,20)
    b.TextColor3 = Color3.new(1,1,1)
    b.Text = text.." : OFF"
    b.Font = Enum.Font.GothamBold
    b.TextSize = 13
    Instance.new("UICorner",b)
    return b
end

local espButton = makeButton("ESP",45)
local highlightButton = makeButton("HIGHLIGHT",85)
local noclipButton = makeButton("NOCLIP",125)
local speedButton = makeButton("SPEED",165)

local status = Instance.new("TextLabel",frame)
status.Position = UDim2.fromOffset(5,215)
status.Size = UDim2.new(1,-10,0,25)
status.BackgroundTransparency = 1
status.Text = "Procurando mulher..."
status.TextColor3 = Color3.new(1,1,1)
status.TextSize = 12

local espGui
local highlight

local function getGirl()
    return workspace:FindFirstChild("girl")
        or workspace:FindFirstChild("GirlBody")
end

local function getPart(girl)
    if girl:IsA("BasePart") then return girl end
    return girl:FindFirstChild("HumanoidRootPart")
        or girl:FindFirstChildWhichIsA("BasePart",true)
end

espButton.MouseButton1Click:Connect(function()
    espOn = not espOn
    espButton.Text = "ESP : "..(espOn and "ON" or "OFF")
end)

highlightButton.MouseButton1Click:Connect(function()
    highlightOn = not highlightOn
    highlightButton.Text = "HIGHLIGHT : "..(highlightOn and "ON" or "OFF")
end)

noclipButton.MouseButton1Click:Connect(function()
    noclipOn = not noclipOn
    noclipButton.Text = "NOCLIP : "..(noclipOn and "ON" or "OFF")
end)

speedButton.MouseButton1Click:Connect(function()
    speedOn = not speedOn
    speedButton.Text = "SPEED : "..(speedOn and "ON" or "OFF")

    local char = player.Character
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")

    if humanoid then
        humanoid.WalkSpeed = speedOn and boostedSpeed or normalSpeed
    end
end)

-- NOCLIP COM RESTAURAÇÃO DE COLISÃO
RunService.Stepped:Connect(function()
    local char = player.Character
    if not char then return end

    for _,part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            if noclipOn then
                if originalCollision[part] == nil then
                    originalCollision[part] = part.CanCollide
                end

                if part.Name == "LeftFoot"
                or part.Name == "RightFoot" then
                    part.CanCollide = true
                else
                    part.CanCollide = false
                end
            elseif originalCollision[part] ~= nil then
                part.CanCollide = originalCollision[part]
                originalCollision[part] = nil
            end
        end
    end
end)

player.CharacterAdded:Connect(function(char)
    originalCollision = {}
    local humanoid = char:WaitForChild("Humanoid")
    if speedOn then
        humanoid.WalkSpeed = boostedSpeed
    end
end)

-- ESP / HIGHLIGHT
RunService.RenderStepped:Connect(function()
    local girl = getGirl()
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")

    if not girl or not root then
        status.Text = "Procurando mulher..."
        if espGui then espGui:Destroy() espGui=nil end
        if highlight then highlight:Destroy() highlight=nil end
        return
    end

    local part = getPart(girl)
    if not part then return end

    if espOn then
        if not espGui or espGui.Adornee ~= part then
            if espGui then espGui:Destroy() end

            espGui = Instance.new("BillboardGui")
            espGui.Size = UDim2.fromOffset(180,35)
            espGui.AlwaysOnTop = true
            espGui.Adornee = part
            espGui.Parent = gui

            local txt = Instance.new("TextLabel",espGui)
            txt.Name = "Distance"
            txt.Size = UDim2.fromScale(1,1)
            txt.BackgroundTransparency = 1
            txt.TextColor3 = Color3.fromRGB(255,30,30)
            txt.TextStrokeTransparency = 0
            txt.Font = Enum.Font.GothamBold
            txt.TextSize = 14
        end

        local distance = (root.Position-part.Position).Magnitude
        espGui.Distance.Text = "MULHER | "..math.floor(distance).."m"
    elseif espGui then
        espGui:Destroy()
        espGui=nil
    end

    if highlightOn then
        if not highlight or highlight.Parent ~= girl then
            if highlight then highlight:Destroy() end
            highlight = Instance.new("Highlight")
            highlight.FillColor = Color3.fromRGB(255,0,0)
            highlight.OutlineColor = Color3.new(1,1,1)
            highlight.Parent = girl
        end
    elseif highlight then
        highlight:Destroy()
        highlight=nil
    end

    status.Text = "Mulher detectada!"
end)
