
--========================================================
-- SHAWARMA HUB FLIGHT V7
-- StarterPlayer > StarterPlayerScripts > LocalScript
--========================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local old = playerGui:FindFirstChild("ShawarmaHubFlight")
if old then old:Destroy() end

local flying = false
local locked = false
local speed = 65

local character, humanoid, root
local attachment, linearVelocity, flightConnection

local function getCharacter(char)
    character = char or player.Character or player.CharacterAdded:Wait()
    humanoid = character:WaitForChild("Humanoid")
    root = character:WaitForChild("HumanoidRootPart")
end

getCharacter()

-- LUXURY SHAWARMA COLOURS
local DARK = Color3.fromRGB(25, 17, 14)
local BROWN = Color3.fromRGB(75, 43, 27)
local GOLD = Color3.fromRGB(218, 166, 91)
local CREAM = Color3.fromRGB(255, 231, 194)
local MUTED = Color3.fromRGB(178, 145, 112)
local GREEN = Color3.fromRGB(111, 190, 124)
local RED = Color3.fromRGB(177, 69, 55)

local function make(class, properties, parent)
    local obj = Instance.new(class)
    for key, value in pairs(properties) do
        obj[key] = value
    end
    obj.Parent = parent
    return obj
end

-- GUI
local gui = make("ScreenGui", {
    Name = "ShawarmaHubFlight",
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling
}, playerGui)

local main = make("Frame", {
    Name = "Main",
    Size = UDim2.fromOffset(330, 270),
    Position = UDim2.new(0.5, -165, 0.5, -135),
    BackgroundColor3 = DARK,
    BorderSizePixel = 0
}, gui)

make("UICorner", {CornerRadius = UDim.new(0, 18)}, main)
make("UIStroke", {Color = GOLD, Thickness = 1.7}, main)

make("UIGradient", {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(53, 35, 25)),
        ColorSequenceKeypoint.new(1, DARK)
    }),
    Rotation = 35
}, main)

-- HEADER
local header = make("Frame", {
    Size = UDim2.new(1, 0, 0, 70),
    BackgroundColor3 = BROWN,
    BorderSizePixel = 0
}, main)

make("UICorner", {CornerRadius = UDim.new(0, 18)}, header)

make("TextLabel", {
    Size = UDim2.new(1, -110, 0, 31),
    Position = UDim2.fromOffset(16, 8),
    BackgroundTransparency = 1,
    Text = "SHAWARMA HUB",
    TextColor3 = CREAM,
    Font = Enum.Font.GothamBlack,
    TextSize = 19,
    TextXAlignment = Enum.TextXAlignment.Left
}, header)

make("TextLabel", {
    Size = UDim2.new(1, -110, 0, 18),
    Position = UDim2.fromOffset(17, 39),
    BackgroundTransparency = 1,
    Text = "✦  FLIGHT EXPERIENCE",
    TextColor3 = GOLD,
    Font = Enum.Font.GothamMedium,
    TextSize = 10,
    TextXAlignment = Enum.TextXAlignment.Left
}, header)

local function headerButton(text, x, color)
    local b = make("TextButton", {
        Size = UDim2.fromOffset(34, 32),
        Position = UDim2.new(1, x, 0, 18),
        BackgroundColor3 = color,
        Text = text,
        TextColor3 = CREAM,
        Font = Enum.Font.GothamBold,
        TextSize = 16
    }, header)

    make("UICorner", {CornerRadius = UDim.new(0, 8)}, b)
    make("UIStroke", {Color = GOLD, Transparency = 0.6}, b)
    return b
end

local lockButton = headerButton("🔓", -82, DARK)
local closeButton = headerButton("×", -42, RED)

-- STATUS
make("TextLabel", {
    Size = UDim2.new(1, -34, 0, 18),
    Position = UDim2.fromOffset(17, 82),
    BackgroundTransparency = 1,
    Text = "FLIGHT STATUS",
    TextColor3 = MUTED,
    Font = Enum.Font.GothamBold,
    TextSize = 10,
    TextXAlignment = Enum.TextXAlignment.Left
}, main)

local status = make("TextLabel", {
    Size = UDim2.new(1, -34, 0, 25),
    Position = UDim2.fromOffset(17, 101),
    BackgroundTransparency = 1,
    Text = "●  FLIGHT OFF",
    TextColor3 = RED,
    Font = Enum.Font.GothamBold,
    TextSize = 15,
    TextXAlignment = Enum.TextXAlignment.Left
}, main)

-- ONE FLIGHT BUTTON
local toggle = make("TextButton", {
    Size = UDim2.new(1, -34, 0, 49),
    Position = UDim2.fromOffset(17, 139),
    BackgroundColor3 = GOLD,
    Text = "✈   ACTIVATE FLIGHT",
    TextColor3 = DARK,
    Font = Enum.Font.GothamBlack,
    TextSize = 15
}, main)

make("UICorner", {CornerRadius = UDim.new(0, 11)}, toggle)

-- SPEED BUTTON
local speedButton = make("TextButton", {
    Size = UDim2.new(1, -34, 0, 38),
    Position = UDim2.fromOffset(17, 198),
    BackgroundColor3 = BROWN,
    Text = "FLIGHT SPEED  •  " .. speed,
    TextColor3 = CREAM,
    Font = Enum.Font.GothamBold,
    TextSize = 12
}, main)

make("UICorner", {CornerRadius = UDim.new(0, 9)}, speedButton)
make("UIStroke", {Color = GOLD, Transparency = 0.65}, speedButton)

make("TextLabel", {
    Size = UDim2.new(1, 0, 0, 18),
    Position = UDim2.fromOffset(0, 245),
    BackgroundTransparency = 1,
    Text = "✦  SHAWARMA HUB FLIGHT  ✦",
    TextColor3 = MUTED,
    Font = Enum.Font.GothamMedium,
    TextSize = 9
}, main)

-- MINIMISED ICON
local reopen = make("TextButton", {
    Size = UDim2.fromOffset(62, 62),
    Position = UDim2.new(0, 18, 0.5, -31),
    BackgroundColor3 = BROWN,
    Text = "🌯",
    TextSize = 29,
    Visible = false
}, gui)

make("UICorner", {CornerRadius = UDim.new(1, 0)}, reopen)
make("UIStroke", {Color = GOLD, Thickness = 2}, reopen)

closeButton.MouseButton1Click:Connect(function()
    main.Visible = false
    reopen.Visible = true
end)

reopen.MouseButton1Click:Connect(function()
    main.Visible = true
    reopen.Visible = false
end)

-- DRAGGING
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

UIS.InputChanged:Connect(function(input)
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

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

lockButton.MouseButton1Click:Connect(function()
    locked = not locked
    lockButton.Text = locked and "🔒" or "🔓"
end)

-- FLIGHT CLEANUP
local function stopFlight()
    flying = false

    if flightConnection then
        flightConnection:Disconnect()
        flightConnection = nil
    end

    if linearVelocity then
        linearVelocity:Destroy()
        linearVelocity = nil
    end

    if attachment then
        attachment:Destroy()
        attachment = nil
    end

    if humanoid then
        humanoid.AutoRotate = true
    end

    toggle.Text = "✈   ACTIVATE FLIGHT"
    toggle.BackgroundColor3 = GOLD
    toggle.TextColor3 = DARK
    status.Text = "●  FLIGHT OFF"
    status.TextColor3 = RED
end

-- START FLIGHT
local function startFlight()
    getCharacter()

    if not root or not humanoid or humanoid.Health <= 0 then
        return
    end

    flying = true
    humanoid.AutoRotate = false

    attachment = Instance.new("Attachment")
    attachment.Name = "ShawarmaFlightAttachment"
    attachment.Parent = root

    linearVelocity = Instance.new("LinearVelocity")
    linearVelocity.Name = "ShawarmaFlightVelocity"
    linearVelocity.Attachment0 = attachment
    linearVelocity.RelativeTo = Enum.ActuatorRelativeTo.World
    linearVelocity.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector
    linearVelocity.MaxForce = 1000000
    linearVelocity.VectorVelocity = Vector3.zero
    linearVelocity.Parent = root

    toggle.Text = "■   STOP FLYING"
    toggle.BackgroundColor3 = RED
    toggle.TextColor3 = CREAM
    status.Text = "●  FLIGHT ACTIVE"
    status.TextColor3 = GREEN

    flightConnection = RunService.Heartbeat:Connect(function()
        if not flying or not root or not root.Parent
        or not humanoid or humanoid.Health <= 0 then
            stopFlight()
            return
        end

        local camera = workspace.CurrentCamera
        local move = humanoid.MoveDirection

        -- Camera pitch adds vertical movement when moving forward/back.
        local cameraForward = camera.CFrame.LookVector
        local flatForward = Vector3.new(
            cameraForward.X, 0, cameraForward.Z
        )

        if flatForward.Magnitude > 0.001 then
            flatForward = flatForward.Unit
        end

        local forwardAmount = move:Dot(flatForward)
        local vertical = cameraForward.Y * forwardAmount

        -- No jump button required.
        -- Joystick/WASD + camera tilt controls all flight.
        local direction = Vector3.new(move.X, vertical, move.Z)

        if direction.Magnitude > 1 then
            direction = direction.Unit
        end

        linearVelocity.VectorVelocity = direction * speed
    end)
end

toggle.MouseButton1Click:Connect(function()
    if flying then
        stopFlight()
    else
        startFlight()
    end
end)

speedButton.MouseButton1Click:Connect(function()
    speed += 15
    if speed > 125 then
        speed = 35
    end
    speedButton.Text = "FLIGHT SPEED  •  " .. speed
end)

player.CharacterAdded:Connect(function(newCharacter)
    stopFlight()
    character = newCharacter
    humanoid = newCharacter:WaitForChild("Humanoid")
    root = newCharacter:WaitForChild("HumanoidRootPart")
end)

print("SHAWARMA HUB FLIGHT V7 loaded")
