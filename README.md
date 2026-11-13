
--================================================
-- SHAWARMA HUB | COMPLETE FLY GUI
--================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local oldGui = playerGui:FindFirstChild("ShawarmaHub")
if oldGui then oldGui:Destroy() end

local flying = false
local speed = 65
local character, humanoid, root
local velocity
local flightConnection

-- CHARACTER
local function loadCharacter()
    character = player.Character or player.CharacterAdded:Wait()
    humanoid = character:WaitForChild("Humanoid")
    root = character:WaitForChild("HumanoidRootPart")
end

loadCharacter()

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "ShawarmaHub"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local main = Instance.new("Frame")
main.Name = "Main"
main.Size = UDim2.fromOffset(280, 155)
main.Position = UDim2.new(0.5, -140, 0.5, -77)
main.BackgroundColor3 = Color3.fromRGB(29, 21, 18)
main.BorderSizePixel = 0
main.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 15)
corner.Parent = main

local outline = Instance.new("UIStroke")
outline.Color = Color3.fromRGB(224, 145, 75)
outline.Thickness = 2
outline.Parent = main

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 48)
header.BackgroundColor3 = Color3.fromRGB(75, 43, 27)
header.BorderSizePixel = 0
header.Parent = main

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 15)
headerCorner.Parent = header

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -15, 1, 0)
title.Position = UDim2.fromOffset(12, 0)
title.BackgroundTransparency = 1
title.Text = "🌯  SHAWARMA HUB"
title.TextColor3 = Color3.fromRGB(255, 224, 180)
title.Font = Enum.Font.GothamBold
title.TextSize = 19
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.new(1, -30, 0, 55)
toggle.Position = UDim2.fromOffset(15, 75)
toggle.BackgroundColor3 = Color3.fromRGB(190, 111, 48)
toggle.Text = "✈  ACTIVATE FLY"
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 17
toggle.AutoButtonColor = true
toggle.Parent = main

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 10)
toggleCorner.Parent = toggle

-- DRAG GUI
local dragging = false
local dragStart
local startPosition

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPosition = main.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- STOP FLIGHT
local function stopFlight()
    flying = false

    if flightConnection then
        flightConnection:Disconnect()
        flightConnection = nil
    end

    if velocity then
        velocity:Destroy()
        velocity = nil
    end

    toggle.Text = "✈  ACTIVATE FLY"
    toggle.BackgroundColor3 = Color3.fromRGB(190, 111, 48)
end

-- START FLIGHT
local function startFlight()
    loadCharacter()

    if not root or not humanoid or humanoid.Health <= 0 then
        return
    end

    flying = true

    velocity = Instance.new("BodyVelocity")
    velocity.Name = "ShawarmaFlightVelocity"
    velocity.MaxForce = Vector3.new(1000000, 1000000, 1000000)
    velocity.P = 15000
    velocity.Velocity = Vector3.zero
    velocity.Parent = root

    toggle.Text = "■  STOP FLYING"
    toggle.BackgroundColor3 = Color3.fromRGB(160, 57, 48)

    flightConnection = RunService.Heartbeat:Connect(function()
        if not flying or not root or not root.Parent
        or not humanoid or humanoid.Health <= 0 then
            stopFlight()
            return
        end

        local camera = workspace.CurrentCamera
        local move = humanoid.MoveDirection
        local vertical = 0

        -- Camera pitch controls vertical movement.
        if move.Magnitude > 0.05 then
            vertical = camera.CFrame.LookVector.Y * move.Magnitude
        end

        -- PC controls.
        if UIS:IsKeyDown(Enum.KeyCode.Space) or humanoid.Jump then
            vertical += 1
        end

        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
            vertical -= 1
        end

        local direction = move + Vector3.new(0, vertical, 0)

        if direction.Magnitude > 1 then
            direction = direction.Unit
        end

        velocity.Velocity = direction * speed
    end)
end

toggle.MouseButton1Click:Connect(function()
    if flying then
        stopFlight()
    else
        startFlight()
    end
end)

-- RESPAWN SUPPORT
player.CharacterAdded:Connect(function(newCharacter)
    stopFlight()
    character = newCharacter
    humanoid = newCharacter:WaitForChild("Humanoid")
    root = newCharacter:WaitForChild("HumanoidRootPart")
end)

print("Shawarma Hub loaded successfully!")
