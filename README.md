
-- SHAWARMA HUB
-- Flight GUI for your own Roblox game

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local flying = false
local flySpeed = 50
local connection
local velocity
local attachment
local orientation

local gui = Instance.new("ScreenGui")
gui.Name = "ShawarmaHub"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(270, 245)
main.Position = UDim2.new(0.5, -135, 0.5, -122)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
main.Parent = gui

Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(145, 70, 255)
stroke.Thickness = 2
stroke.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundColor3 = Color3.fromRGB(35, 25, 50)
title.Text = "🌯 SHAWARMA HUB"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 19
title.Font = Enum.Font.GothamBold
title.Parent = main

Instance.new("UICorner", title).CornerRadius = UDim.new(0, 10)

local function createButton(name, text, position, color)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.85, 0, 0, 38)
    button.Position = position
    button.BackgroundColor3 = color
    button.Text = text
    button.TextColor3 = Color3.new(1, 1, 1)
    button.TextSize = 14
    button.Font = Enum.Font.GothamBold
    button.Parent = main
    Instance.new("UICorner", button).CornerRadius = UDim.new(0, 8)
    return button
end

local activate = createButton(
    "Activate",
    "ACTIVATE FLY",
    UDim2.new(0.075, 0, 0, 60),
    Color3.fromRGB(120, 55, 220)
)

local stop = createButton(
    "Stop",
    "STOP FLY",
    UDim2.new(0.075, 0, 0, 60),
    Color3.fromRGB(190, 55, 65)
)
stop.Visible = false

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0.85, 0, 0, 25)
speedLabel.Position = UDim2.new(0.075, 0, 0, 110)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "FLY SPEED"
speedLabel.TextColor3 = Color3.fromRGB(200, 190, 220)
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextSize = 13
speedLabel.Parent = main

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0.85, 0, 0, 35)
speedBox.Position = UDim2.new(0.075, 0, 0, 140)
speedBox.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
speedBox.Text = "50"
speedBox.PlaceholderText = "Enter speed"
speedBox.TextColor3 = Color3.new(1, 1, 1)
speedBox.Font = Enum.Font.Gotham
speedBox.TextSize = 14
speedBox.Parent = main
Instance.new("UICorner", speedBox).CornerRadius = UDim.new(0, 8)

local setSpeed = createButton(
    "SetSpeed",
    "SET SPEED",
    UDim2.new(0.075, 0, 0, 185),
    Color3.fromRGB(65, 110, 180)
)

local function stopFlying()
    flying = false

    if connection then
        connection:Disconnect()
        connection = nil
    end

    if velocity then
        velocity:Destroy()
        velocity = nil
    end

    if orientation then
        orientation:Destroy()
        orientation = nil
    end

    if attachment then
        attachment:Destroy()
        attachment = nil
    end

    activate.Visible = true
    stop.Visible = false
end

local function startFlying()
    local character = player.Character
    local root = character and character:FindFirstChild("HumanoidRootPart")
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")

    if not root or not humanoid or flying then return end

    flying = true

    attachment = Instance.new("Attachment")
    attachment.Parent = root

    velocity = Instance.new("LinearVelocity")
    velocity.Attachment0 = attachment
    velocity.RelativeTo = Enum.ActuatorRelativeTo.World
    velocity.MaxForce = 1000000
    velocity.VectorVelocity = Vector3.zero
    velocity.Parent = root

    orientation = Instance.new("AlignOrientation")
    orientation.Attachment0 = attachment
    orientation.Mode = Enum.OrientationAlignmentMode.OneAttachment
    orientation.MaxTorque = 1000000
    orientation.Parent = root

    activate.Visible = false
    stop.Visible = true

    connection = RunService.RenderStepped:Connect(function()
        if not flying or not root.Parent then
            stopFlying()
            return
        end

        local camera = workspace.CurrentCamera
        local direction = Vector3.zero

        if UIS:IsKeyDown(Enum.KeyCode.W) then
            direction += camera.CFrame.LookVector
        end
        if UIS:IsKeyDown(Enum.KeyCode.S) then
            direction -= camera.CFrame.LookVector
        end
        if UIS:IsKeyDown(Enum.KeyCode.A) then
            direction -= camera.CFrame.RightVector
        end
        if UIS:IsKeyDown(Enum.KeyCode.D) then
            direction += camera.CFrame.RightVector
        end
        if UIS:IsKeyDown(Enum.KeyCode.Space) then
            direction += Vector3.yAxis
        end
        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
            direction -= Vector3.yAxis
        end

        velocity.VectorVelocity =
            direction.Magnitude > 0
            and direction.Unit * flySpeed
            or Vector3.zero

        orientation.CFrame = camera.CFrame
    end)
end

activate.MouseButton1Click:Connect(startFlying)
stop.MouseButton1Click:Connect(stopFlying)

setSpeed.MouseButton1Click:Connect(function()
    local number = tonumber(speedBox.Text)
    if number then
        flySpeed = math.clamp(number, 1, 200)
        speedBox.Text = tostring(flySpeed)
        setSpeed.Text = "SPEED SET!"
        task.delay(1, function()
            if setSpeed.Parent then
                setSpeed.Text = "SET SPEED"
            end
        end)
    end
end)

player.CharacterAdded:Connect(function()
    stopFlying()
end)

-- Draggable GUI
local dragging = false
local dragStart, startPosition

title.InputBegan:Connect(function(input)
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
