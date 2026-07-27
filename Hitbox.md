local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = (gethui and gethui()) or game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

local HitEvent = ReplicatedStorage:FindFirstChild("HitEvent")
if not HitEvent then
    HitEvent = Instance.new("RemoteEvent")
    HitEvent.Name = "HitEvent"
    HitEvent.Parent = ReplicatedStorage
end

if CoreGui:FindFirstChild("XellwareHitbox") then CoreGui.XellwareHitbox:Destroy() end

local RegularHitboxEnabled = false
local SmartHitboxEnabled = false
local AntiStandEnabled = true 
local GroundHitEnabled = false
local HitboxSize = 8.5 
local HitboxTrans = 0.0 

local AttackPauseUntil = 0
local HitRegisterPauseTime = 0.40
local hitCharacters = {}
local guiVisible = true

local function applyRetroTheme(element, isButton)
    element.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    local stroke = Instance.new("UIStroke", element)
    stroke.Color = Color3.fromRGB(200, 200, 200)
    stroke.Thickness = 2
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    local corner = Instance.new("UICorner", element)
    corner.CornerRadius = UDim.new(0, 0)
    
    if element:IsA("TextLabel") or element:IsA("TextButton") then
        element.TextColor3 = Color3.new(1, 1, 1)
        element.Font = Enum.Font.Code 
    end
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XellwareHitbox"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

local OpenButton = Instance.new("TextButton", ScreenGui)
OpenButton.Position = UDim2.new(0, 20, 0, 20)
OpenButton.Size = UDim2.new(0, 90, 0, 40)
OpenButton.Text = "[ HITBOX ]"
OpenButton.TextSize = 14
OpenButton.Visible = false
applyRetroTheme(OpenButton, true)

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -180)
MainFrame.Size = UDim2.new(0, 350, 0, 360)
MainFrame.Active = true
MainFrame.Draggable = true 
applyRetroTheme(MainFrame, false)

local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 35)
TopBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
TopBar.BorderSizePixel = 0

local Title = Instance.new("TextLabel", TopBar)
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 15, 0, 0)
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Text = "XELLWARE - HITBOX"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.Code
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left

local CloseButton = Instance.new("TextButton", TopBar)
CloseButton.Position = UDim2.new(1, -35, 0, 5)
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Text = "X"
applyRetroTheme(CloseButton, true)
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)

local Container = Instance.new("ScrollingFrame", MainFrame)
Container.BackgroundTransparency = 1
Container.Position = UDim2.new(0, 10, 0, 45)
Container.Size = UDim2.new(1, -20, 1, -55)
Container.ScrollBarThickness = 6
local layout = Instance.new("UIListLayout", Container)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 10)

local function createToggle(text, defaultState, callback)
    local btn = Instance.new("TextButton", Container)
    btn.Size = UDim2.new(1, -10, 0, 38)
    btn.Text = text .. (defaultState and " [ON]" or " [OFF]")
    btn.TextSize = 15
    applyRetroTheme(btn, true)
    if defaultState then btn.TextColor3 = Color3.fromRGB(100, 255, 100) end
    btn.MouseButton1Click:Connect(function() callback(btn) end)
    return btn
end

local function createSlider(text, minVal, maxVal, defaultVal, callback)
    local frame = Instance.new("Frame", Container)
    frame.Size = UDim2.new(1, -10, 0, 50)
    applyRetroTheme(frame, false)
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, -10, 0, 20)
    label.Position = UDim2.new(0, 8, 0, 2)
    label.BackgroundTransparency = 1
    label.Text = text .. ": " .. tostring(defaultVal)
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.Code
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    local track = Instance.new("Frame", frame)
    track.Size = UDim2.new(1, -20, 0, 12)
    track.Position = UDim2.new(0, 10, 0, 28)
    track.BackgroundColor3 = Color3.new(0, 0, 0)
    local stroke = Instance.new("UIStroke", track)
    stroke.Color = Color3.new(1, 1, 1)
    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = Color3.new(1, 1, 1)
    fill.BorderSizePixel = 0
    local btn = Instance.new("TextButton", track)
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    local dragging = false
    local function update(input)
        local pos = math.clamp((input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(pos, 0, 1, 0)
        local value = math.floor((minVal + ((maxVal - minVal) * pos)) * 100) / 100 
        label.Text = text .. ": " .. tostring(value)
        callback(value)
    end
    btn.InputBegan:Connect(function(inp) if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then dragging = true; update(inp) end end)
    UserInputService.InputEnded:Connect(function(inp) if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then dragging = false end end)
    UserInputService.InputChanged:Connect(function(inp) if dragging and (inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch) then update(inp) end end)
end

CloseButton.MouseButton1Click:Connect(function() MainFrame.Visible = false; OpenButton.Visible = true end)
OpenButton.MouseButton1Click:Connect(function() MainFrame.Visible = true; OpenButton.Visible = false end)

createToggle("Regular Hitbox ESP", false, function(btn)
    RegularHitboxEnabled = not RegularHitboxEnabled
    btn.Text = "Regular Hitbox ESP" .. (RegularHitboxEnabled and " [ON]" or " [OFF]")
    btn.TextColor3 = RegularHitboxEnabled and Color3.fromRGB(100, 255, 100) or Color3.new(1, 1, 1)
end)

createToggle("Smart Hitbox (99% Hit Rate)", false, function(btn)
    SmartHitboxEnabled = not SmartHitboxEnabled
    btn.Text = "Smart Hitbox (99% Hit Rate)" .. (SmartHitboxEnabled and " [ON]" or " [OFF]")
    btn.TextColor3 = SmartHitboxEnabled and Color3.fromRGB(100, 255, 100) or Color3.new(1, 1, 1)
end)

createSlider("Hitbox Size", 1.0, 25.0, 8.5, function(val) HitboxSize = val end)
createSlider("Hide Visuals", 0.0, 1.0, 0.0, function(val) HitboxTrans = val end)

createToggle("Anti-Stand (Fix Collisions)", true, function(btn)
    AntiStandEnabled = not AntiStandEnabled
    btn.Text = "Anti-Stand (Fix Collisions)" .. (AntiStandEnabled and " [ON]" or " [OFF]")
    btn.TextColor3 = AntiStandEnabled and Color3.fromRGB(100, 255, 100) or Color3.new(1, 1, 1)
end)

createToggle("Hit Grounded (Ragdoll)", false, function(btn)
    GroundHitEnabled = not GroundHitEnabled
    btn.Text = "Hit Grounded (Ragdoll)" .. (GroundHitEnabled and " [ON]" or " [OFF]")
    btn.TextColor3 = GroundHitEnabled and Color3.fromRGB(100, 255, 100) or Color3.new(1, 1, 1)
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and (input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.RightShift) then
        guiVisible = not guiVisible
        ScreenGui.Enabled = guiVisible
    end
    if not gameProcessed and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
        local now = os.clock()
        if now >= AttackPauseUntil then
            AttackPauseUntil = now + HitRegisterPauseTime
            if SmartHitboxEnabled then
                table.clear(hitCharacters)
                local startTime = os.clock()
                local lastHrpCFrame = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.CFrame
                local connection
                connection = RunService.Heartbeat:Connect(function()
                    if os.clock() - startTime >= math.min(HitRegisterPauseTime, 0.4) then
                        if connection then connection:Disconnect() end return
                    end
                    local character = LocalPlayer.Character
                    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
                    local currentCFrame = character.HumanoidRootPart.CFrame
                    local distanceMoved = 0
                    local boxCFrame = currentCFrame
                    if lastHrpCFrame then
                        distanceMoved = (currentCFrame.Position - lastHrpCFrame.Position).Magnitude
                        if distanceMoved > 0.01 then boxCFrame = CFrame.lookAt(lastHrpCFrame.Position, currentCFrame.Position) * CFrame.new(0, 0, -distanceMoved / 2) end
                    end
                    local boxSize = Vector3.new(HitboxSize, HitboxSize, HitboxSize + distanceMoved)
                    lastHrpCFrame = currentCFrame 
                    local overlapParams = OverlapParams.new()
                    overlapParams.FilterDescendantsInstances = {character}
                    overlapParams.FilterType = Enum.RaycastFilterType.Exclude
                    for _, part in ipairs(workspace:GetPartBoundsInBox(boxCFrame, boxSize, overlapParams)) do
                        local model = part:FindFirstAncestorOfClass("Model")
                        if model and model ~= character and model:FindFirstChild("Humanoid") then
                            local hum = model.Humanoid
                            local state = hum:GetState()
                            local isRagdolled = hum.PlatformStand or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown
                            if hum.Health > 0 and not hitCharacters[model] then
                                if not isRagdolled or GroundHitEnabled then
                                    hitCharacters[model] = true 
                                    HitEvent:FireServer(model)
                                end
                            end
                        end
                    end
                end)
            end
        end
    end
end)

RunService.Stepped:Connect(function()
    if RegularHitboxEnabled and AntiStandEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                local head = player.Character:FindFirstChild("Head")
                if hrp then hrp.CanCollide = false end
                if head then head.CanCollide = false end
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            local hrp = player.Character.HumanoidRootPart
            if RegularHitboxEnabled and player.Character.Humanoid.Health > 0 then
                hrp.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
                hrp.Transparency = 1 
                local outline = hrp:FindFirstChild("HitboxOutline")
                if not outline then
                    outline = Instance.new("SelectionBox")
                    outline.Name = "HitboxOutline"
                    outline.Adornee = hrp
                    outline.LineThickness = 0.05 
                    outline.Color3 = Color3.new(1, 0, 0)
                    outline.Parent = hrp
                end
                outline.Transparency = HitboxTrans 
            else
                hrp.Size = Vector3.new(2, 2, 1) 
                hrp.Transparency = 1
                if hrp:FindFirstChild("HitboxOutline") then hrp.HitboxOutline:Destroy() end
            end
        end
    end
end)
