local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ================= CẤU TRÚC UI ================= --
local ScreenGui = Instance.new("ScreenGui")
-- Nếu chạy qua executor có thể dùng CoreGui để giấu UI, nếu chạy trong game studio thì dùng PlayerGui
ScreenGui.Parent = (RunService:IsStudio() or not pcall(function() return CoreGui end)) and LocalPlayer:WaitForChild("PlayerGui") or CoreGui
ScreenGui.Name = "RedAnimationHub"

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 450, 0, 300)
MainFrame.Position = UDim2.new(0.5, -225, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 0, 0)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(255, 30, 30)
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

-- Tiêu đề
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -40, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "🔴 RED ANIMATION HUB"
Title.TextColor3 = Color3.fromRGB(255, 50, 50)
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 20
Title.Parent = MainFrame

-- Nút thu nhỏ (Minimize)
local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0, 40, 0, 40)
MinBtn.Position = UDim2.new(1, -40, 0, 0)
MinBtn.BackgroundTransparency = 1
MinBtn.Text = "-"
MinBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinBtn.TextSize = 25
MinBtn.Font = Enum.Font.GothamBold
MinBtn.Parent = MainFrame

-- Khu vực chứa nút
local Container = Instance.new("ScrollingFrame")
Container.Size = UDim2.new(1, -20, 1, -50)
Container.Position = UDim2.new(0, 10, 0, 40)
Container.BackgroundTransparency = 1
Container.ScrollBarThickness = 4
Container.Parent = MainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 10)
UIListLayout.Parent = Container

-- Hàm tạo nút (Button Template)
local function createButton(text, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.BackgroundColor3 = Color3.fromRGB(40, 0, 0)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 200, 200)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.Parent = Container

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = btn

    btn.MouseButton1Click:Connect(function()
        -- Hiệu ứng bóng click
        local tween = game:GetService("TweenService"):Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(150, 0, 0)})
        tween:Play()
        task.wait(0.1)
        game:GetService("TweenService"):Create(btn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(40, 0, 0)}):Play()
        callback()
    end)
    return btn
end

-- ================= LOGIC TÍNH NĂNG ================= --

-- 1. Super Speed 100k
local speedEnabled = false
createButton("⚡ Super Speed (100k) [BẬT/TẮT]", function()
    speedEnabled = not speedEnabled
    if speedEnabled then
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = 100000
        end
    else
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = 16
        end
    end
end)

-- 2. Fly (Bay)
local flying = false
createButton("🦅 Fly [BẬT/TẮT]", function()
    flying = not flying
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart

    if flying then
        local bv = Instance.new("BodyVelocity")
        bv.Name = "FlyVelocity"
        bv.MaxForce = Vector3.new(100000, 100000, 100000)
        bv.Parent = hrp
        local bg = Instance.new("BodyGyro")
        bg.Name = "FlyGyro"
        bg.MaxTorque = Vector3.new(100000, 100000, 100000)
        bg.Parent = hrp

        RunService:BindToRenderStep("FlyStep", Enum.RenderPriority.Camera.Value, function()
            if not flying then return end
            local moveDir = char.Humanoid.MoveDirection
            bv.Velocity = moveDir * 150 -- Tốc độ bay
            bg.CFrame = Camera.CFrame
        end)
    else
        RunService:UnbindFromRenderStep("FlyStep")
        if hrp:FindFirstChild("FlyVelocity") then hrp.FlyVelocity:Destroy() end
        if hrp:FindFirstChild("FlyGyro") then hrp.FlyGyro:Destroy() end
    end
end)

-- 3. ESP (Nhìn xuyên tường bằng Highlight)
local espEnabled = false
createButton("👁️ ESP (Nhìn Xuyên Tường) [BẬT/TẮT]", function()
    espEnabled = not espEnabled
    if espEnabled then
        for _, v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and not v.Character:FindFirstChild("RedESP") then
                local hl = Instance.new("Highlight")
                hl.Name = "RedESP"
                hl.FillColor = Color3.fromRGB(255, 0, 0)
                hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                hl.Parent = v.Character
            end
        end
    else
        for _, v in pairs(Players:GetPlayers()) do
            if v.Character and v.Character:FindFirstChild("RedESP") then
                v.Character.RedESP:Destroy()
            end
        end
    end
end)

-- 4. Lock Aim (Khóa mục tiêu)
local aimlock = false
local function getClosestPlayer()
    local closestDist = math.huge
    local target = nil
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Head") and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
            local pos, onScreen = Camera:WorldToViewportPoint(v.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                if dist < closestDist then
                    closestDist = dist
                    target = v
                end
            end
        end
    end
    return target
end

createButton("🎯 Lock Aim (Chuột phải) [BẬT/TẮT]", function()
    aimlock = not aimlock
end)

RunService.RenderStepped:Connect(function()
    if aimlock and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)

-- 5. Touch Kill (Chỉ hoạt động hoàn toàn nếu game thiết lập sát thương Client-side)
local touchKill = false
createButton("💀 Touch Kill [BẬT/TẮT]", function()
    touchKill = not touchKill
    local char = LocalPlayer.Character
    if not char then return end

    for _, part in pairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Touched:Connect(function(hit)
                if touchKill and hit.Parent and hit.Parent:FindFirstChild("Humanoid") then
                    local targetPlayer = Players:GetPlayerFromCharacter(hit.Parent)
                    if targetPlayer and targetPlayer ~= LocalPlayer then
                        -- Sát thương ở Client. Nếu game của bạn FE, cần thay bằng RemoteEvent báo lên Server!
                        hit.Parent.Humanoid.Health = 0
                    end
                end
            end)
        end
    end
end)

-- 6. Server Hop
createButton("🌐 Server Hop", function()
    -- Cách đơn giản nhất để đổi server là Teleport lại chính game này (hệ thống sẽ cố gắng tìm server khác)
    TeleportService:Teleport(game.PlaceId, LocalPlayer)
end)

-- ================= ANIMATION THU NHỎ ================= --
local isMinimized = false
MinBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    MinBtn.Text = isMinimized and "+" or "-"
    Container.Visible = not isMinimized
    local targetSize = isMinimized and UDim2.new(0, 450, 0, 40) or UDim2.new(0, 450, 0, 300)
    
    local tween = game:GetService("TweenService"):Create(MainFrame, TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = targetSize})
    tween:Play()
end)

-- DRAG UI (Cho phép kéo thả bảng)
local dragging, dragInput, dragStart, startPos
Title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
Title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
