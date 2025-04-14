-- Painel Futurista com ESP, Aimbot e FOV por ChatGPT

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local camera = workspace.CurrentCamera
local lp = Players.LocalPlayer

-- CONFIGURAÇÃO DO FOV
local fovRadius = 150

-- UI
local gui = Instance.new("ScreenGui", lp:WaitForChild("PlayerGui"))
gui.Name = "PainelESP"

local toggleUIBtn = Instance.new("TextButton", gui)
toggleUIBtn.Size = UDim2.new(0, 30, 0, 30)
toggleUIBtn.Position = UDim2.new(0, 10, 0, 10)
toggleUIBtn.Text = "-"
toggleUIBtn.BackgroundColor3 = Color3.fromRGB(120, 0, 180)
toggleUIBtn.TextColor3 = Color3.new(1, 1, 1)
toggleUIBtn.Font = Enum.Font.GothamBold
toggleUIBtn.TextSize = 20
Instance.new("UICorner", toggleUIBtn)

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 320, 0, 450)
frame.Position = UDim2.new(0.3, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 0, 50)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", frame).Color = Color3.fromRGB(140, 0, 255)
frame.Active = true
frame.Draggable = true

toggleUIBtn.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
    toggleUIBtn.Text = frame.Visible and "-" or "+"
end)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "Painel Futurista"
title.TextColor3 = Color3.fromRGB(255, 200, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.BackgroundTransparency = 1

local function criarBotao(txt, y)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0.8, 0, 0, 40)
    btn.Position = UDim2.new(0.1, 0, y, 0)
    btn.Text = txt
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.BackgroundColor3 = Color3.fromRGB(70, 0, 120)
    btn.TextColor3 = Color3.new(1, 1, 1)
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 10)
    return btn
end

local toggleESPBtn = criarBotao("ESP: ON", 0.2)
local toggleBoxBtn = criarBotao("Box: ON", 0.32)
local toggleSkeletonBtn = criarBotao("Skeleton: OFF", 0.44)
local toggleNameBtn = criarBotao("Names: OFF", 0.56)
local keybindBtn = criarBotao("Aimbot Key: E", 0.68)
local aimbotModeBtn = criarBotao("Aimbot Mode: Hold", 0.80)

local colorBox = Instance.new("TextButton", frame)
colorBox.Size = UDim2.new(0, 30, 0, 30)
colorBox.Position = UDim2.new(1, -40, 0, 10)
colorBox.Text = ""
colorBox.BackgroundColor3 = Color3.fromRGB(255, 0, 255)
Instance.new("UICorner", colorBox)

-- ESTADO
local aimbotKey = Enum.KeyCode.E
local aimbotMode = "Hold"
local settings = {
    espEnabled = true,
    showBox = true,
    showSkeleton = false,
    showName = false,
    useRGB = true
}

-- COR RGB
local function getColor()
    return settings.useRGB and Color3.fromHSV(tick() % 5 / 5, 1, 1) or colorBox.BackgroundColor3
end

colorBox.MouseButton1Click:Connect(function()
    settings.useRGB = not settings.useRGB
    colorBox.BackgroundColor3 = getColor()
end)

toggleESPBtn.MouseButton1Click:Connect(function()
    settings.espEnabled = not settings.espEnabled
    toggleESPBtn.Text = "ESP: " .. (settings.espEnabled and "ON" or "OFF")
end)

toggleBoxBtn.MouseButton1Click:Connect(function()
    settings.showBox = not settings.showBox
    toggleBoxBtn.Text = "Box: " .. (settings.showBox and "ON" or "OFF")
end)

toggleSkeletonBtn.MouseButton1Click:Connect(function()
    settings.showSkeleton = not settings.showSkeleton
    toggleSkeletonBtn.Text = "Skeleton: " .. (settings.showSkeleton and "ON" or "OFF")
end)

toggleNameBtn.MouseButton1Click:Connect(function()
    settings.showName = not settings.showName
    toggleNameBtn.Text = "Names: " .. (settings.showName and "ON" or "OFF")
end)

keybindBtn.MouseButton1Click:Connect(function()
    keybindBtn.Text = "Press key or mouse..."
    local conn
    conn = UserInputService.InputBegan:Connect(function(input, gpe)
        if not gpe then
            if input.UserInputType == Enum.UserInputType.Keyboard then
                aimbotKey = input.KeyCode
                keybindBtn.Text = "Aimbot Key: " .. input.KeyCode.Name
            elseif input.UserInputType == Enum.UserInputType.MouseButton1 then
                aimbotKey = "MouseButton1"
                keybindBtn.Text = "Aimbot Key: Left Click"
            elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
                aimbotKey = "MouseButton2"
                keybindBtn.Text = "Aimbot Key: Right Click"
            elseif input.UserInputType == Enum.UserInputType.MouseButton3 then
                aimbotKey = "MouseButton3"
                keybindBtn.Text = "Aimbot Key: Middle Click"
            end
            conn:Disconnect()
        end
    end)
end)

aimbotModeBtn.MouseButton1Click:Connect(function()
    aimbotMode = (aimbotMode == "Hold") and "Toggle" or "Hold"
    aimbotModeBtn.Text = "Aimbot Mode: " .. aimbotMode
end)

-- FOV CIRCLE
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Thickness = 2
fovCircle.Filled = false
fovCircle.Visible = true
fovCircle.Radius = fovRadius

-- ESP e AIMBOT
local drawings = {}
local aiming = false

RunService.RenderStepped:Connect(function()
    local mousePos = UserInputService:GetMouseLocation()
    fovCircle.Position = Vector2.new(mousePos.X, mousePos.Y)

    for _, v in pairs(drawings) do for _, d in pairs(v) do d.Visible = false end end
    if not settings.espEnabled then return end

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            local head = p.Character:FindFirstChild("Head")
            local hrp = p.Character:FindFirstChild("HumanoidRootPart")
            if not hrp then continue end
            local pos, onScreen = camera:WorldToViewportPoint(hrp.Position)
            if onScreen then
                local color = getColor()
                drawings[p] = drawings[p] or {}
                if settings.showBox and head then
                    local leg = p.Character:FindFirstChild("RightLowerLeg") or p.Character:FindFirstChild("LeftLowerLeg") or hrp
                    local headPos, onScreenHead = camera:WorldToViewportPoint(head.Position)
                    local legPos, onScreenLeg = camera:WorldToViewportPoint(leg.Position)
                    if onScreenHead and onScreenLeg then
                        local height = math.abs(headPos.Y - legPos.Y)
                        local width = height / 2
                        local box = drawings[p].box or Drawing.new("Square")
                        box.Visible = true
                        box.Color = color
                        box.Thickness = 2
                        box.Filled = false
                        box.Size = Vector2.new(width, height)
                        box.Position = Vector2.new(pos.X - width / 2, headPos.Y)
                        drawings[p].box = box
                    end
                end
                if settings.showName and head then
                    local label = drawings[p].name or Drawing.new("Text")
                    label.Visible = true
                    label.Text = p.Name
                    label.Color = color
                    label.Size = 14
                    label.Center = true
                    local headPos = camera:WorldToViewportPoint(head.Position + Vector3.new(0, 1.5, 0))
                    label.Position = Vector2.new(headPos.X, headPos.Y)
                    drawings[p].name = label
                end
            end
        end
    end
end)

UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    local match = (
        (typeof(aimbotKey) == "EnumItem" and input.KeyCode == aimbotKey)
        or (aimbotKey == "MouseButton1" and input.UserInputType == Enum.UserInputType.MouseButton1)
        or (aimbotKey == "MouseButton2" and input.UserInputType == Enum.UserInputType.MouseButton2)
        or (aimbotKey == "MouseButton3" and input.UserInputType == Enum.UserInputType.MouseButton3)
    )
    if match then
        if aimbotMode == "Hold" then
            aiming = true
        elseif aimbotMode == "Toggle" then
            aiming = not aiming
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, gpe)
    if aimbotMode == "Hold" then
        local match = (
            (typeof(aimbotKey) == "EnumItem" and input.KeyCode == aimbotKey)
            or (aimbotKey == "MouseButton1" and input.UserInputType == Enum.UserInputType.MouseButton1)
            or (aimbotKey == "MouseButton2" and input.UserInputType == Enum.UserInputType.MouseButton2)
            or (aimbotKey == "MouseButton3" and input.UserInputType == Enum.UserInputType.MouseButton3)
        )
        if match then aiming = false end
    end
end)

RunService.RenderStepped:Connect(function()
    if not aiming then return end
    local mousePos = UserInputService:GetMouseLocation()
    local closest, shortest = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            local pos, onScreen = camera:WorldToViewportPoint(p.Character.HumanoidRootPart.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
                if dist < shortest and dist < fovRadius then
                    shortest = dist
                    closest = p
                end
            end
        end
    end
    if closest then
        camera.CFrame = CFrame.new(camera.CFrame.Position, closest.Character.HumanoidRootPart.Position)
    end
end)
