local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- UI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.Position = UDim2.new(0.05, 0, 0.2, 0)
MainFrame.Size = UDim2.new(0, 220, 0, 420)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, -35, 0, 35)
Title.Text = "MAP ADMIN V19"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

local MinBtn = Instance.new("TextButton", MainFrame)
MinBtn.Size = UDim2.new(0, 35, 0, 35)
MinBtn.Position = UDim2.new(1, -35, 0, 0)
MinBtn.Text = "-"
MinBtn.BackgroundColor3 = Color3.fromRGB(60, 20, 20)
MinBtn.TextColor3 = Color3.new(1,1,1)

local minimized = false
MinBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    MainFrame:TweenSize(minimized and UDim2.new(0, 220, 0, 35) or UDim2.new(0, 220, 0, 420), "Out", "Quad", 0.3, true)
    MinBtn.Text = minimized and "+" or "-"
end)

local Content = Instance.new("ScrollingFrame", MainFrame)
Content.Size = UDim2.new(1, 0, 1, -35)
Content.Position = UDim2.new(0, 0, 0, 35)
Content.BackgroundTransparency = 1
Content.CanvasSize = UDim2.new(0, 0, 4, 0)
Instance.new("UIListLayout", Content).Padding = UDim.new(0, 5)

local function createBtn(text, parent, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    btn.TextColor3 = Color3.new(1,1,1)
    local act = false
    btn.MouseButton1Click:Connect(function()
        act = not act
        btn.BackgroundColor3 = act and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
        callback(act)
    end)
    return btn
end

-- === SISTEMA: VOID DROP (LEVAR E ABANDONAR) ===
local voidDropActive = false
local lastSurfacePos = nil

createBtn("VOID DROP (LEVAR/SOLTAR)", Content, function(act)
    voidDropActive = act
    if act then
        -- Salva a posição antes de descer para poder voltar
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            lastSurfacePos = player.Character.HumanoidRootPart.CFrame
        end
        
        task.spawn(function()
            while voidDropActive do
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    local hrp = player.Character.HumanoidRootPart
                    -- Mantém no Void
                    hrp.CFrame = CFrame.new(hrp.Position.X, -1000, hrp.Position.Z)
                    hrp.Velocity = Vector3.new(0,0,0)
                end
                RunService.Heartbeat:Wait()
            end
            
            -- QUANDO DESLIGAR O BOTÃO:
            if player.Character and player.Character:FindFirstChild("Humanoid") then
                -- 1. Força você a soltar qualquer coisa (pular)
                player.Character.Humanoid.Sit = false
                task.wait(0.1)
                -- 2. Te traz de volta para onde você estava antes de descer
                if lastSurfacePos then
                    player.Character.HumanoidRootPart.CFrame = lastSurfacePos + Vector3.new(0, 5, 0)
                end
            end
        end)
    end
end)

-- === OUTRAS OPÇÕES MANTIDAS ===
createBtn("Velocidade 100", Content, function(s) player.Character.Humanoid.WalkSpeed = s and 100 or 16 end)
createBtn("Pulo Infinito", Content, function(s)
    _G.IJ = s
    UserInputService.JumpRequest:Connect(function() if _G.IJ then player.Character.Humanoid:ChangeState("Jumping") end end)
end)
createBtn("Noclip", Content, function(s) _G.NC = s end)
RunService.Stepped:Connect(function()
    if _G.NC and player.Character then
        for _, v in pairs(player.Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = false end end
    end
end)
createBtn("Voo", Content, function(s)
    _G.Fly = s
    if s then
        local bv = Instance.new("BodyVelocity", player.Character.HumanoidRootPart)
        bv.MaxForce = Vector3.new(1,1,1) * math.huge
        task.spawn(function()
            while _G.Fly do bv.Velocity = camera.CFrame.LookVector * 100 task.wait() end
            bv:Destroy()
        end)
    end
end)

local SelectedTarget = nil
local PlayerPanel = Instance.new("Frame", ScreenGui)
PlayerPanel.Size = UDim2.new(0, 180, 0, 200)
PlayerPanel.Position = UDim2.new(0.25, 0, 0.2, 0)
PlayerPanel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
PlayerPanel.Visible = false
local PScroll = Instance.new("ScrollingFrame", PlayerPanel)
PScroll.Size = UDim2.new(1, 0, 1, 0)
PScroll.BackgroundTransparency = 1
Instance.new("UIListLayout", PScroll)

createBtn("Lista de Alvos", Content, function(s)
    PlayerPanel.Visible = s
    if s then
        for _, v in pairs(PScroll:GetChildren()) do if v:IsA("TextButton") then v:Destroy() end end
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= player then
                local b = Instance.new("TextButton", PScroll)
                b.Size = UDim2.new(1, 0, 0, 25)
                b.Text = p.Name
                b.MouseButton1Click:Connect(function() SelectedTarget = p b.Text = "OK: "..p.Name end)
            end
        end
    end
end)

createBtn("Visualizar Alvo", Content, function(s)
    camera.CameraSubject = (s and SelectedTarget and SelectedTarget.Character) and SelectedTarget.Character.Humanoid or player.Character.Humanoid
end)

createBtn("FLING BOLA (KILL)", Content, function(s)
    _G.Fling = s
    task.spawn(function()
        while _G.Fling do
            if SelectedTarget and SelectedTarget.Character and SelectedTarget.Character:FindFirstChild("HumanoidRootPart") then
                local bola = workspace:FindFirstChild("bola") or workspace:FindFirstChild("Bola")
                if bola then
                    local p = bola:IsA("BasePart") and bola or bola:FindFirstChildWhichIsA("BasePart")
                    if p then
                        local bp = p:FindFirstChild("FPos") or Instance.new("BodyPosition", p)
                        bp.Name = "FPos"; bp.MaxForce = Vector3.new(1,1,1)*math.huge; bp.P = 20000
                        bp.Position = SelectedTarget.Character.HumanoidRootPart.Position
                        local bav = p:FindFirstChild("FSpin") or Instance.new("BodyAngularVelocity", p)
                        bav.Name = "FSpin"; bav.MaxTorque = Vector3.new(1,1,1)*math.huge; bav.AngularVelocity = Vector3.new(0, 10000, 0)
                        p.Velocity = Vector3.new(0,1000,0); p.CanCollide = true
                    end
                end
            end
            task.wait(0.01)
        end
    end)
end)
