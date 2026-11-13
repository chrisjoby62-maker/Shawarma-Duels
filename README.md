
--====================================================
--             🌯 SHAWARMA HUB V4
--====================================================

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

--====================================================
-- GUI CREATION
--====================================================

local gui = Instance.new("ScreenGui")
gui.Name = "ShawarmaHub"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Name = "Main"
main.Size = UDim2.fromOffset(300, 310)
main.Position = UDim2.new(0.5, -150, 0.5, -155)
main.BackgroundColor3 = Color3.fromRGB(19, 16, 22)
main.BorderSizePixel = 0
main.ClipsDescendants = true
main.Parent = gui

Instance.new("UICorner", main).CornerRadius = UDim.new(0, 16)

local outline = Instance.new("UIStroke")
outline.Color = Color3.fromRGB(145, 75, 220)
outline.Thickness = 2
outline.Parent = main

--====================================================
-- SHAWARMA BACKGROUND
--====================================================

local background = Instance.new("ImageLabel")
background.Name = "ShawarmaBackground"
background.Size = UDim2.fromScale(1, 1)
background.Position = UDim2.fromScale(0, 0)
background.BackgroundTransparency = 1

-- Upload your shawarma background image to Roblox
-- and replace the placeholder below with its asset ID.
background.Image = "rbxassetid://YOUR_IMAGE_ID"
background.ImageTransparency = 0.55
background.ScaleType = Enum.ScaleType.Crop
background.ZIndex = 1
background.Parent = main

local overlay = Instance.new("Frame")
overlay.Size = UDim2.fromScale(1, 1)
overlay.BackgroundColor3 = Color3.fromRGB(15, 10, 20)
overlay.BackgroundTransparency = 0.20
overlay.BorderSizePixel = 0
overlay.ZIndex = 2
overlay.Parent = main

--====================================================
-- BUTTON CREATOR
--====================================================

local function createButton(parent, name, text, size, pos, color)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = size
    button.Position = pos
    button.BackgroundColor3 = color
    button.Text = text
    button.TextColor3 = Color3.new(1, 1, 1)
    button.TextSize = 14
    button.Font = Enum.Font.GothamBold
    button.BorderSizePixel = 0
    button.AutoButtonColor = true
    button.ZIndex = 5
    button.Parent = parent

    Instance.new("UICorner", button).CornerRadius =
        UDim.new(0, 9)

    return button
end

--====================================================
-- HEADER
--====================================================

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 55)
header.BackgroundColor3 = Color3.fromRGB(35, 23, 43)
header.BackgroundTransparency = 0.05
header.BorderSizePixel = 0
header.ZIndex = 4
header.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(0.58, 0, 1, 0)
title.Position = UDim2.fromOffset(13, 0)
title.BackgroundTransparency = 1
title.Text = "🌯 SHAWARMA HUB"
title.TextColor3 = Color3.fromRGB(255, 235, 215)
title.Font = Enum.Font.GothamBlack
title.TextSize = 17
title.TextXAlignment = Enum.TextXAlignment.Left
title.ZIndex = 5
title.Parent = header

local lockButton = createButton(
    header, "Lock", "🔓",
    UDim2.fromOffset(34, 30),
    UDim2.new(1, -80, 0, 12),
    Color3.fromRGB(90, 60, 110)
)

local closeButton = createButton(
    header, "Close", "✕",
    UDim2.fromOffset(34, 30),
    UDim2.new(1, -39, 0, 12),
    Color3.fromRGB(170, 55, 65)
)

--====================================================
-- STATUS
--====================================================

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -30, 0, 25)
status.Position = UDim2.fromOffset(15, 62)
status.BackgroundTransparency = 1
status.Text = "● FLIGHT SYSTEM READY"
status.TextColor3 = Color3.fromRGB(150, 220, 160)
status.Font = Enum.Font.GothamBold
status.TextSize = 12
status.TextXAlignment = Enum.TextXAlignment.Left
status.ZIndex = 5
status.Parent = main

--====================================================
-- ACTIVATE / STOP
--====================================================

local activate = createButton(
    main, "ActivateFly", "✈  ACTIVATE FLY",
    UDim2.new(1, -36, 0, 43),
    UDim2.fromOffset(18, 100),
    Color3.fromRGB(115, 55, 190)
)

local stop = createButton(
    main, "StopFly", "■  STOP FLY",
    UDim2.new(1, -36, 0, 43),
    UDim2.fromOffset(18, 100),
    Color3.fromRGB(185, 55, 65)
)

stop.Visible = false

--====================================================
-- SPEED SETTINGS
--====================================================

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1, -36, 0, 25)
speedLabel.Position = UDim2.fromOffset(18, 157)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "FLIGHT SPEED"
speedLabel.TextColor3 = Color3.fromRGB(230, 205, 240)
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextSize = 12
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.ZIndex = 5
speedLabel.Parent = main

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(1, -36, 0, 38)
speedBox.Position = UDim2.fromOffset(18, 184)
speedBox.BackgroundColor3 = Color3.fromRGB(35, 30, 42)
speedBox.Text = "50"
speedBox.PlaceholderText = "Enter speed"
speedBox.TextColor3 = Color3.new(1, 1, 1)
speedBox.Font = Enum.Font.GothamBold
speedBox.TextSize = 15
speedBox.ClearTextOnFocus = false
speedBox.ZIndex = 5
speedBox.Parent = main

Instance.new("UICorner", speedBox).CornerRadius =
    UDim.new(0, 8)

local setSpeed = createButton(
    main, "SetSpeed", "UPDATE SPEED",
    UDim2.new(1, -36, 0, 36),
    UDim2.fromOffset(18, 231),
    Color3.fromRGB(85, 65, 145)
)

local footer = Instance.new("TextLabel")
footer.Size = UDim2.new(1, -30, 0, 20)
footer.Position = UDim2.fromOffset(15, 278)
footer.BackgroundTransparency = 1
footer.Text = "SHAWARMA HUB  •  V4"
footer.TextColor3 = Color3.fromRGB(190, 155, 205)
footer.Font = Enum.Font.GothamBold
footer.TextSize = 10
footer.ZIndex = 5
footer.Parent = main

--====================================================
-- MINIMIZED BUTTON
--====================================================

local mini = createButton(
    gui, "MiniButton", "🌯",
    UDim2.fromOffset(58, 58),
    UDim2.new(0, 20, 0.5, -29),
    Color3.fromRGB(100, 45, 155)
)

mini.TextSize = 26
mini.Visible = false

closeButton.MouseButton1Click:Connect(function()
    main.Visible = false
    mini.Visible = true
end)

mini.MouseButton1Click:Connect(function()
    main.Visible = true
    mini.Visible = false
end)

--====================================================
-- LOCK SYSTEM
--====================================================

lockButton.MouseButton1Click:Connect(function()
    locked = not locked

    lockButton.Text = locked and "🔒" or "🔓"

    lockButton.BackgroundColor3 = locked
        and Color3.fromRGB(170, 65, 65)
        or Color3.fromRGB(90, 60, 110)
end)

--====================================================
-- DRAG SYSTEM
--====================================================

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

--====================================================
-- FLIGHT SYSTEM - NO SPINNING
--====================================================

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

    local character = player.Character
    local humanoid = character and
        character:FindFirstChildOfClass("Humanoid")

    if humanoid then
        humanoid.AutoRotate = true
        humanoid:ChangeState(
            Enum.HumanoidStateType.GettingUp
        )
    end

    activate.Visible = true
    stop.Visible = false

    status.Text = "● FLIGHT SYSTEM READY"
    status.TextColor3 = Color3.fromRGB(150, 220, 160)
end

local function startFlying()
    if flying then return end

    local character = player.Character
    local root = character and
        character:FindFirstChild("HumanoidRootPart")

    local humanoid = character and
        character:FindFirstChildOfClass("Humanoid")

    if not root or not humanoid then return end

    flying = true
    humanoid.AutoRotate = false

    attachment = Instance.new("Attachment")
    attachment.Parent = root

    velocity = Instance.new("LinearVelocity")
    velocity.Attachment0 = attachment
    velocity.RelativeTo = Enum.ActuatorRelativeTo.World
    velocity.VelocityConstraintMode =
        Enum.VelocityConstraintMode.Vector
    velocity.MaxForce = 1000000
    velocity.VectorVelocity = Vector3.zero
    velocity.Parent = root

    orientation = Instance.new("AlignOrientation")
    orientation.Attachment0 = attachment
    orientation.Mode =
        Enum.OrientationAlignmentMode.OneAttachment
    orientation.MaxTorque = 1000000
    orientation.Responsiveness = 15
    orientation.Parent = root

    activate.Visible = false
    stop.Visible = true

    status.Text = "● FLIGHT ACTIVE"
    status.TextColor3 = Color3.fromRGB(160, 240, 170)

    connection = RunService.Heartbeat:Connect(function()
        if not flying or not root.Parent then
            stopFlying()
            return
        end

        local camera = workspace.CurrentCamera
        local moveDirection = humanoid.MoveDirection

        -- Normal Roblox joystick and WASD movement
        local direction = Vector3.zero

        if moveDirection.Magnitude > 0.05 then
            direction = moveDirection

            -- Preserve horizontal movement without
            -- following camera pitch or camera roll.
            direction = Vector3.new(
                direction.X,
                0,
                direction.Z
            )
        end

        -- PC vertical movement
        if UIS:IsKeyDown(Enum.KeyCode.Space) then
            direction += Vector3.yAxis
        end

        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
            direction -= Vector3.yAxis
        end

        if direction.Magnitude > 0 then
            direction = direction.Unit
        end

        velocity.VectorVelocity = direction * flySpeed

        -- Face movement direction, never camera rotation.
        -- This prevents camera-induced spinning.
        local horizontal = Vector3.new(
            moveDirection.X,
            0,
            moveDirection.Z
        )

        if horizontal.Magnitude > 0.05 then
            orientation.CFrame = CFrame.lookAt(
                root.Position,
                root.Position + horizontal.Unit
            )
        end
    end)
end

activate.MouseButton1Click:Connect(startFlying)
stop.MouseButton1Click:Connect(stopFlying)

--====================================================
-- SPEED UPDATE
--====================================================

setSpeed.MouseButton1Click:Connect(function()
    local value = tonumber(speedBox.Text)

    if value then
        flySpeed = math.clamp(value, 1, 200)
        speedBox.Text = tostring(flySpeed)
        setSpeed.Text = "SPEED UPDATED!"

        task.delay(1, function()
            if setSpeed.Parent then
                setSpeed.Text = "UPDATE SPEED"
            end
        end)
    else
        speedBox.Text = tostring(flySpeed)
    end
end)

--====================================================
-- RESPAWN CLEANUP
--====================================================

player.CharacterAdded:Connect(function()
    stopFlying()
end)

print("🌯 SHAWARMA HUB V4 LOADED")
