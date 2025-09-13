-- Menu Anti-Lag no estilo "hub"
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local player = Players.LocalPlayer
local gui = player:WaitForChild("PlayerGui")

-- salva valores originais de iluminação
local orig = {
    GlobalShadows = Lighting.GlobalShadows,
    Brightness = Lighting.Brightness,
    FogEnd = Lighting.FogEnd,
    Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient,
}
local anti = false

local function setAntiLag(on)
    anti = on
    if on then
        pcall(function()
            Lighting.GlobalShadows = false
            Lighting.Brightness = 1
            Lighting.FogEnd = 1000
            Lighting.Ambient = Color3.fromRGB(128,128,128)
            Lighting.OutdoorAmbient = Color3.fromRGB(100,100,100)
        end)
    else
        pcall(function()
            Lighting.GlobalShadows = orig.GlobalShadows
            Lighting.Brightness = orig.Brightness
            Lighting.FogEnd = orig.FogEnd
            Lighting.Ambient = orig.Ambient
            Lighting.OutdoorAmbient = orig.OutdoorAmbient
        end)
    end
    for _,o in ipairs(Workspace:GetDescendants()) do
        if o:IsA("ParticleEmitter") or o:IsA("Trail") or o:IsA("Beam")
        or o:IsA("Fire") or o:IsA("Smoke") or o:IsA("Sparkles")
        or o:IsA("PointLight") or o:IsA("SurfaceLight") or o:IsA("SpotLight") then
            pcall(function() o.Enabled = not on end)
        end
        if o:IsA("Decal") or o:IsA("Texture") then
            pcall(function() o.Transparency = on and 1 or 0 end)
        end
        if o:IsA("BasePart") then
            pcall(function() if on then o.Material = Enum.Material.SmoothPlastic end end)
        end
    end

    -- ===== Otimizações extras =====
    -- Esconder nomes de NPCs
    for _, model in ipairs(Workspace:GetDescendants()) do
        if model:IsA("Model") and model:FindFirstChild("Humanoid") then
            local hum = model:FindFirstChildOfClass("Humanoid")
            if hum and not Players:GetPlayerFromCharacter(model) then
                -- se não for personagem de jogador (é NPC)
                pcall(function() hum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None end)
            end
        end
    end

    -- "Skin" super simples em todos os jogadores (menos você)
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            for _, part in ipairs(plr.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    pcall(function()
                        if on then
                            part.Material = Enum.Material.SmoothPlastic
                            part.Color = Color3.fromRGB(0,0,0) -- tudo preto
                        else
                            -- não dá pra restaurar a cor exata sem salvar antes,
                            -- então aqui deixamos só SmoothPlastic
                            part.Material = Enum.Material.SmoothPlastic
                        end
                    end)
                end
            end
        end
    end

    warn(on and "[Anti-Lag] Ativado" or "[Anti-Lag] Desativado")
end

-- ===== UI no estilo de hub =====
if gui:FindFirstChild("FPSHub") then gui.FPSHub:Destroy() end
local screen = Instance.new("ScreenGui", gui)
screen.Name = "FPSHub"
screen.ResetOnSpawn = false

local main = Instance.new("Frame", screen)
main.Size = UDim2.new(0,300,0,180)
main.Position = UDim2.new(0.5,-150,0.5,-90)
main.BackgroundColor3 = Color3.fromRGB(25,25,25)
main.BackgroundTransparency = 0.2
main.BorderSizePixel = 0
main.AnchorPoint = Vector2.new(0.5,0.5)
main.Active = true
main.Draggable = true    -- permite arrastar o menu

local uicorner = Instance.new("UICorner", main)
uicorner.CornerRadius = UDim.new(0,12)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Text = "FPS Hub - Anti Lag"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.new(1,1,1)

local btn = Instance.new("TextButton", main)
btn.Size = UDim2.new(1,-40,0,50)
btn.Position = UDim2.new(0,20,0,60)
btn.Text = "Ativar Anti-Lag"
btn.Font = Enum.Font.Gotham
btn.TextSize = 18
btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
btn.TextColor3 = Color3.new(1,1,1)

local cornerBtn = Instance.new("UICorner", btn)
cornerBtn.CornerRadius = UDim.new(0,8)

btn.MouseButton1Click:Connect(function()
    setAntiLag(not anti)
    btn.Text = anti and "Desativar Anti-Lag" or "Ativar Anti-Lag"
end)

-- ===== Tela Esticada (efeito) =====
local stretched = false
local stretchFrame = Instance.new("Frame", screen)
stretchFrame.Size = UDim2.new(1.1,0,1,0)  -- 10% mais largo
stretchFrame.Position = UDim2.new(-0.05,0,0,0) -- centraliza
stretchFrame.BackgroundColor3 = Color3.new(0,0,0)
stretchFrame.BorderSizePixel = 0
stretchFrame.Visible = false

local viewport = Instance.new("ViewportFrame", stretchFrame)
viewport.Size = UDim2.new(1,0,1,0)
viewport.BackgroundTransparency = 1
viewport.CurrentCamera = workspace.CurrentCamera

-- Atualiza a câmera do viewport
workspace.CurrentCamera:GetPropertyChangedSignal("CFrame"):Connect(function()
    viewport.CurrentCamera = workspace.CurrentCamera
end)

local function setTelaEsticada(on)
    stretched = on
    stretchFrame.Visible = on
end

-- Botão Tela Esticada
local btnStretch = Instance.new("TextButton", main)
btnStretch.Size = UDim2.new(1,-40,0,50)
btnStretch.Position = UDim2.new(0,20,0,120)
btnStretch.Text = "Tela Esticada"
btnStretch.Font = Enum.Font.Gotham
btnStretch.TextSize = 18
btnStretch.BackgroundColor3 = Color3.fromRGB(40,40,40)
btnStretch.TextColor3 = Color3.new(1,1,1)

local cornerBtn2 = Instance.new("UICorner", btnStretch)
cornerBtn2.CornerRadius = UDim.new(0,8)

btnStretch.MouseButton1Click:Connect(function()
    setTelaEsticada(not stretched)
    btnStretch.Text = stretched and "Desativar Tela Esticada" or "Tela Esticada"
end)
