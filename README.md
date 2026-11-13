
--==================================================
-- SHAWARMA HUB V5 | FLIGHT GUI
-- Place in StarterPlayer > StarterPlayerScripts
--==================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local flying = false
local locked = false
local minimized = false
local speed = 65

local character, humanoid, root
local velocity, gyro
local flightConnection

local oldGui = playerGui:FindFirstChild("ShawarmaHub")
if oldGui then oldGui:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "ShawarmaHub"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local function new(class, props, parent)
    local obj = Instance.new(class)
    for k, v in pairs(props) do
        obj[k] = v
    end
    obj.Parent = parent
    return obj
end

local main = new("Frame", {
    Size = UDim2.fromOffset(300, 270),
    Position = UDim2.new(0.5, -150, 0.5, -135),
    BackgroundColor3 = Color3.fromRGB(24, 19, 17),
    BorderSizePixel = 0
}, gui)

new("UICorner", {CornerRadius = UDim.new(0, 14)}, main)
new("UIStroke", {
    Color = Color3.fromRGB(214, 133, 66),
    Thickness = 2
}, main)

local header = new("Frame", {
    Size = UDim2.new(1, 0, 0, 48),
    BackgroundColor3 = Color3.fromRGB(54, 32, 22),
    BorderSizePixel = 0
}, main)
new("UICorner", {CornerRadius = UDim.new(0, 14)}, header)

new("TextLabel", {
    Size = UDim2.new(1, -100, 1, 0),
    Position = UDim2.fromOffset(12, 0),
    BackgroundTransparency = 1,
    Text = "🌯 SHAWARMA HUB",
    TextColor3 = Color3.fromRGB(255, 220, 170),
    TextSize = 19,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left
}, header)

local lockButton = new("TextButton", {
    Size = UDim2.fromOffset(34, 30),
    Position = UDim2.new(1, -78, 0, 9),
    BackgroundColor3 = Color3.fromRGB(91, 57, 37),
    Text = "🔓",
    TextSize = 16,
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1)
}, header)
new("UICorner", {CornerRadius = UDim.new(0, 8)}, lockButton)

local closeButton = new("TextButton", {
    Size = UDim2.fromOffset(34, 30),
    Position = UDim2.new(1, -39, 0, 9),
    BackgroundColor3 = Color3.fromRGB(160, 57, 45),
    Text = "—",
    TextSize = 19,
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1)
}, header)
new("UICorner", {CornerRadius = UDim.new(0, 8)}, closeButton)

local status = new("TextLabel", {
    Size = UDim2.new(1, -30, 0, 28),
    Position = UDim2.fromOffset(15, 60),
    BackgroundTransparency = 1,
    Text = "● FLIGHT DISABLED",
    TextColor3 = Color3.fromRGB(220, 120, 100),
    TextSize = 13,
    Font = Enum.Font.GothamBold
}, main)

local function makeButton(text, y, color)
    local button = new("TextButton", {
        Size = UDim2.new(1, -40, 0, 42),
        Position = UDim2.fromOffset(20, y),
        BackgroundColor3 = color,
        Text = text,
        TextColor3 = Color3.new(1, 1, 1),
        TextSize = 15,
        Font = Enum.Font.GothamBold,
        AutoButtonColor = true
    }, main)
    new("UICorner", {CornerRadius = UDim.new(0, 9)}, button)
    return button
end

local flyButton = makeButton("✈  ACTIVATE FLIGHT", 100, Color3.fromRGB(170, 95, 43))
local speedButton = makeButton("SPEED: 65  (CLICK TO CHANGE)", 151, Color3.fromRGB(75, 55, 43))
local stopButton = makeButton("■  STOP FLIGHT", 202, Color3.fromRGB(135, 55, 45))

local reopen = new("TextButton", {
    Size = UDim2.fromOffset(62, 62),
    Position = UDim2.new(0, 18, 0.5, -31),
    BackgroundColor3 = Color3.fromRGB(54, 32, 22),
    Text = "🌯",
    TextSize = 30,
    Visible = false,
    TextColor3 = Color3.new(1, 1, 1)
}, gui)
new("UICorner", {CornerRadius = UDim.new(1, 0)}, reopen)
new("UIStroke", {Color = Color3.fromRGB(214, 133, 66), Thickness = 2}, reopen)

-- Dragging
local dragging = false
local dragStart, startPosition

header.InputBegan:Connect(function(input)
    if locked then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPosition = main.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if not dragging or locked then return end
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
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

lockButton.MouseButton1Click:Connect(function()
    locked = not locked
    lockButton.Text = locked and "🔒" or "🔓"
end)

closeButton.MouseButton1Click:Connect(function()
    main.Visible = false
    reopen.Visible = true
    minimized = true
end)

reopen.MouseButton1Click:Connect(function()
    main.Visible = true
    reopen.Visible = false
    minimized = false
end)

-- Character setup
local function getCharacter()
    character = player.Character or player.CharacterAdded:Wait()
    humanoid = character:WaitForChild("Humanoid")
    root = character:WaitForChild("HumanoidRootPart")
end

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

    if gyro then
        gyro:Destroy()
        gyro = nil
    end

    if humanoid then
        humanoid.PlatformStand = false
        humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
    end

    status.Text = "● FLIGHT DISABLED"
    status.TextColor3 = Color3.fromRGB(220, 120, 100)
    flyButton.Text = "✈  ACTIVATE FLIGHT"
end

local function startFlight()
    getCharacter()

    if not root or not humanoid or humanoid.Health <= 0 then return end

    if velocity then velocity:Destroy() end
    if gyro then gyro:Destroy() end

    flying = true

    velocity = Instance.new("BodyVelocity")
    velocity.Name = "ShawarmaFlightVelocity"
    velocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
    velocity.P = 15000
    velocity.Velocity = Vector3.zero
    velocity.Parent = root

    gyro = Instance.new("BodyGyro")
    gyro.Name = "ShawarmaFlightGyro"
    gyro.MaxTorque = Vector3.new(0, 1e6, 0)
    gyro.P = 12000
    gyro.D = 500
    gyro.CFrame = root.CFrame
    gyro.Parent = root

    humanoid.PlatformStand = false

    status.Text = "● FLIGHT ACTIVE"
    status.TextColor3 = Color3.fromRGB(110, 225, 140)
    flyButton.Text = "✈  FLIGHT ACTIVATED"

    flightConnection = RunService.Heartbeat:Connect(function()
        if not flying or not root or not root.Parent then
            stopFlight()
            return
        end

        local camera = workspace.CurrentCamera
        local move = humanoid.MoveDirection
        local horizontal = Vector3.new(move.X, 0, move.Z)

        local vertical = 0

        -- Look up/down while moving to control flight height.
        if horizontal.Magnitude > 0.05 then
            vertical = camera.CFrame.LookVector.Y
                * horizontal.Magnitude
        end

        -- Standard keyboard controls.
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            vertical = vertical + 1
        end

        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
            vertical = vertical - 1
        end

        -- Mobile jump button.
        if humanoid.Jump then
            vertical = vertical + 1
        end

        local direction = horizontal + Vector3.new(0, vertical, 0)

        if direction.Magnitude > 1 then
            direction = direction.Unit
        end

        velocity.Velocity = direction * speed

        -- Turn only horizontally to prevent spinning.
        if horizontal.Magnitude > 0.05 then
            gyro.CFrame = CFrame.lookAt(
                root.Position,
                root.Position + horizontal.Unit
            )
        end
    end)
end

flyButton.MouseButton1Click:Connect(function()
    if flying then return end
    startFlight()
end)

stopButton.MouseButton1Click:Connect(stopFlight)

speedButton.MouseButton1Click:Connect(function()
    speed = speed + 15
    if speed > 125 then
        speed = 35
    end
    speedButton.Text = "SPEED: " .. speed .. "  (CLICK TO CHANGE)"
end)

player.CharacterAdded:Connect(function()
    stopFlight()
    task.wait(0.5)
    getCharacter()
end)

getCharacter()
