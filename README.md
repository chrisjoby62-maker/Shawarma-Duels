
-- SHAWARMA HUB V2
-- For your own Roblox Studio game

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local flying = false
local locked = false
local flySpeed = 50

local connection
local attachment
local velocity
local orientation
local mobileUp = false
local mobileDown = false

local gui = Instance.new("ScreenGui")
gui.Name = "ShawarmaHub"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- MAIN WINDOW
local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(280, 310)
main.Position = UDim2.new(0.5, -140, 0.5, -155)
main.BackgroundColor3 = Color3.fromRGB(19, 19, 25)
main.BorderSizePixel = 0
main.Parent = gui

Instance.new("UICorner", main).CornerRadius = UDim.new(0, 14)

local outline = Instance.new("UIStroke")
outline.Color = Color3.fromRGB(145, 65, 255)
outline.Thickness = 2
outline.Parent = main

-- HEADER
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 48)
header.BackgroundColor3 = Color3.fromRGB(35, 25, 50)
header.BorderSizePixel = 0
header.Parent = main

Instance.new("UICorner", header).CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(0.55, 0, 1, 0)
title.Position = UDim2.fromOffset(12, 0)
title.BackgroundTransparency = 1
title.Text = "🌯 SHAWARMA HUB"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 17
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

local function makeButton(parent, name, text, size, position, color)
    local b = Instance.new("TextButton")
    b.Name = name
    b.Size = size
    b.Position = position
    b.BackgroundColor3 = color
    b.Text = text
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.BorderSizePixel = 0
    b.Parent = parent

    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
    return b
end

-- HEADER CONTROLS
local lockButton = makeButton(
    header, "Lock", "🔓",
    UDim2.fromOffset(35, 30),
    UDim2.new(1, -82, 0, 9),
    Color3.fromRGB(75, 60, 100)
)

local closeButton = makeButton(
    header, "Close", "X",
    UDim2.fromOffset(35, 30),
    UDim2.new(1, -42, 0, 9),
    Color3.fromRGB(180, 55, 65)
)

-- FLIGHT BUTTONS
local activate = makeButton(
    main, "Activate", "✈ ACTIVATE FLY",
    UDim2.new(0.86, 0, 0, 40),
    UDim2.new(0.07, 0, 0, 65),
    Color3.fromRGB(120, 55, 220)
)

local stop = makeButton(
    main, "Stop", "■ STOP FLY",
    UDim2.new(0.86, 0, 0, 40),
    UDim2.new(0.07, 0, 0, 65),
    Color3.fromRGB(190, 55, 65)
)
stop.Visible = false

-- SPEED
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0.86, 0, 0, 25)
speedLabel.Position = UDim2.new(0.07, 0, 0, 120)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "FLIGHT SPEED"
speedLabel.TextColor3 = Color3.fromRGB(200, 190, 220)
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextSize = 13
speedLabel.Parent = main

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0.86, 0, 0, 35)
speedBox.Position = UDim2.new(0.07, 0, 0, 150)
speedBox.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
speedBox.Text = "50"
speedBox.TextColor3 = Color3.new(1, 1, 1)
speedBox.Font = Enum.Font.Gotham
speedBox.TextSize = 14
speedBox.ClearTextOnFocus = false
speedBox.Parent = main
Instance.new("UICorner", speedBox).CornerRadius = UDim.new(0, 8)

local setSpeed = makeButton(
    main, "SetSpeed", "SET SPEED",
    UDim2.new(0.86, 0, 0, 35),
    UDim2.new(0.07, 0, 0, 195),
    Color3.fromRGB(65, 110, 180)
)

-- MOBILE FLIGHT CONTROLS
local upButton = makeButton(
    main, "Up", "▲ UP",
    UDim2.new(0.40, 0, 0, 35),
    UDim2.new(0.07, 0, 0, 245),
    Color3.fromRGB(65, 140, 95)
)

local downButton = makeButton(
    main, "Down", "▼ DOWN",
    UDim2.new(0.40, 0, 0, 35),
    UDim2.new(0.53, 0, 0, 245),
    Color3.fromRGB(160, 90, 65)
)

-- MINIMIZED SHAWARMA BUTTON
local mini = makeButton(
    gui, "MiniButton", "🌯",
    UDim2.fromOffset(55, 55),
    UDim2.new(0, 20, 0.5, -27),
    Color3.fromRGB(100, 45, 170)
)
mini.TextSize = 25
mini.Visible = false

-- CLOSE AND REOPEN
closeButton.MouseButton1Click:Connect(function()
    main.Visible = false
    mini.Visible = true
end)

mini.MouseButton1Click:Connect(function()
    main.Visible = true
    mini.Visible = false
end)

-- LOCK / UNLOCK
lockButton.MouseButton1Click:Connect(function()
    locked = not locked
    lockButton.Text = locked and "🔒" or "🔓"
    lockButton.BackgroundColor3 = locked
        and Color3.fromRGB(160, 65, 65)
        or Color3.fromRGB(75, 60, 100)
end)

-- DRAGGING
local dragging = false
local dragStart
local startPosition

header.InputBegan:Connect(function(input)
    if locked then return end

    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPosition = main.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and not locked then
        if input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch then

            local delta = input.Position - dragStart

            main.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- MOBILE BUTTON HOLD
upButton.MouseButton1Down:Connect(function()
    mobileUp = true
end)

upButton.MouseButton1Up:Connect(function()
    mobileUp = false
end)

downButton.MouseButton1Down:Connect(function()
    mobileDown = true
end)

downButton.MouseButton1Up:Connect(function()
    mobileDown = false
end)

UIS.TouchEnded:Connect(function()
    mobileUp = false
    mobileDown = false
end)

-- FLIGHT SYSTEM
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

    mobileUp = false
    mobileDown = false

    activate.Visible = true
    stop.Visible = false
    upButton.Visible = false
    downButton.Visible = false
end

local function startFlying()
    if flying then return end

    local character = player.Character
    local root = character and character:FindFirstChild("HumanoidRootPart")
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")

    if not root or not humanoid then return end

    flying = true
    humanoid.PlatformStand = true

    attachment = Instance.new("Attachment")
    attachment.Parent = root

    velocity = Instance.new("LinearVelocity")
    velocity.Attachment0 = attachment
    velocity.RelativeTo = Enum.ActuatorRelativeTo.World
    velocity.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector
    velocity.MaxForce = 1000000
    velocity.VectorVelocity = Vector3.zero
    velocity.Parent = root

    orientation = Instance.new("AlignOrientation")
    orientation.Attachment0 = attachment
    orientation.Mode = Enum.OrientationAlignmentMode.OneAttachment
    orientation.MaxTorque = 1000000
    orientation.Responsiveness = 20
    orientation.Parent = root

    activate.Visible = false
    stop.Visible = true
    upButton.Visible = true
    downButton.Visible = true

    connection = RunService.Heartbeat:Connect(function()
        if not flying or not root.Parent then
            stopFlying()
            return
        end

        local camera = workspace.CurrentCamera
        local direction = Vector3.zero

        -- Keyboard
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

        -- Vertical controls
        if UIS:IsKeyDown(Enum.KeyCode.Space) or mobileUp then
            direction += Vector3.yAxis
        end

        if UIS:IsKeyDown(Enum.KeyCode.LeftControl)
            or mobileDown then
            direction -= Vector3.yAxis
        end

        if direction.Magnitude > 0 then
            direction = direction.Unit
        end

        velocity.VectorVelocity = direction * flySpeed
        orientation.CFrame = camera.CFrame
    end)
end

activate.MouseButton1Click:Connect(startFlying)
stop.MouseButton1Click:Connect(stopFlying)

-- SET SPEED
setSpeed.MouseButton1Click:Connect(function()
    local value = tonumber(speedBox.Text)

    if value then
        flySpeed = math.clamp(value, 1, 200)
        speedBox.Text = tostring(flySpeed)
        setSpeed.Text = "SPEED SET!"

        task.delay(1, function()
            if setSpeed.Parent then
                setSpeed.Text = "SET SPEED"
            end
        end)
    end
end)

-- RESPAWN
player.CharacterAdded:Connect(function(character)
    stopFlying()
    local humanoid = character:WaitForChild("Humanoid")
    humanoid.PlatformStand = false
end)

-- INITIAL STATE
upButton.Visible = false
downButton.Visible = false

print("SHAWARMA HUB V2 LOADED")
