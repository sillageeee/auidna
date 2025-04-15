--// Tenta carregar a Rayfield com URL alternativa
local success, Rayfield = pcall(function()
   return loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()
end)

if not success or not Rayfield then
   warn("Não foi possível carregar a Rayfield UI. Verifique sua conexão ou troque a URL para a versão compatível com seu executor.")
   return
end

--// Configuração da Rayfield UI
local Window = Rayfield:CreateWindow({
   Name = "Xenz Hub",
   Icon = 0, -- ícone opcional (0 para nenhum ícone)
   LoadingTitle = "Rayfield Interface Suite",
   LoadingSubtitle = "by Xenz Cheats",
   Theme = "Default",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil,
      FileName = "Big Hub"
   },
   Discord = {
      Enabled = false,
   },
   KeySystem = false,
})

--// Variáveis de configuração do Aimbot
local AimbotEnabled = false
local ShowFOV = true
local AimbotFOV = 100
local AimbotSmoothness = 0.1
local WallCheck = true
local TeamCheck = true
local HeadSize = 1

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

--// Cria círculo do FOV usando Drawing (certifique-se que o executor suporte a função Drawing)
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Thickness = 1
fovCircle.Filled = false
fovCircle.Transparency = 0.5
fovCircle.Radius = AimbotFOV
fovCircle.Visible = ShowFOV

RunService.RenderStepped:Connect(function()
    fovCircle.Visible = ShowFOV
    fovCircle.Radius = AimbotFOV
    fovCircle.Position = Vector2.new(Mouse.X, Mouse.Y + 36)
end)

--// Função de visibilidade (WallCheck)
local function IsVisible(part)
    if not WallCheck then return true end
    local origin = Camera.CFrame.Position
    local direction = (part.Position - origin)
    local result = workspace:Raycast(origin, direction, {LocalPlayer.Character, part.Parent})
    return result == nil or result.Instance:IsDescendantOf(part.Parent)
end

--// Função para encontrar o inimigo mais próximo dentro do FOV (usa a "Head" como alvo)
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            if TeamCheck and player.Team == LocalPlayer.Team then
                continue
            end

            local head = player.Character.Head
            local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
            if onScreen and IsVisible(head) then
                local distance = (Vector2.new(Mouse.X, Mouse.Y) - Vector2.new(pos.X, pos.Y)).Magnitude
                if distance < shortestDistance and distance <= AimbotFOV then
                    shortestDistance = distance
                    closestPlayer = head
                end
            end
        end
    end

    return closestPlayer
end

--// Lógica do Aimbot: movimenta a câmera suavemente para mirar no alvo
RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local target = GetClosestPlayer()
        if target then
            local lookVector = (target.Position - Camera.CFrame.Position).Unit
            local newCF = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + lookVector)
            Camera.CFrame = Camera.CFrame:Lerp(newCF, AimbotSmoothness)
        end
    end
end)

--// Criação da aba Aimbot na UI
local AimbotTab = Window:CreateTab("Aimbot", 0)

AimbotTab:CreateToggle({
   Name = "Ativar Aimbot",
   CurrentValue = false,
   Flag = "AimbotEnabled",
   Callback = function(Value)
      AimbotEnabled = Value
   end,
})

AimbotTab:CreateToggle({
   Name = "Mostrar FOV",
   CurrentValue = true,
   Flag = "ShowFOV",
   Callback = function(Value)
      ShowFOV = Value
      fovCircle.Visible = Value
   end,
})

AimbotTab:CreateSlider({
   Name = "Tamanho do FOV",
   Range = {10, 500},
   Increment = 5,
   Suffix = "px",
   CurrentValue = 100,
   Flag = "AimbotFOV",
   Callback = function(Value)
      AimbotFOV = Value
      fovCircle.Radius = Value
   end,
})

AimbotTab:CreateSlider({
   Name = "Suavidade",
   Range = {0, 1},
   Increment = 0.01,
   Suffix = "",
   CurrentValue = 0.1,
   Flag = "AimbotSmoothness",
   Callback = function(Value)
      AimbotSmoothness = Value
   end,
})

AimbotTab:CreateToggle({
   Name = "WallCheck (verifica paredes)",
   CurrentValue = true,
   Flag = "WallCheck",
   Callback = function(Value)
      WallCheck = Value
   end,
})

AimbotTab:CreateToggle({
   Name = "TeamCheck (verifica time)",
   CurrentValue = true,
   Flag = "TeamCheck",
   Callback = function(Value)
      TeamCheck = Value
   end,
})

AimbotTab:CreateSlider({
   Name = "Tamanho da Cabeça",
   Range = {1, 10},
   Increment = 0.5,
   Suffix = "x",
   CurrentValue = 1,
   Flag = "HeadSize",
   Callback = function(Value)
      HeadSize = Value
      -- Ajuste o tamanho das cabeças dos jogadores (exceto o local)
      for _, player in pairs(Players:GetPlayers()) do
         if player.Character and player.Character:FindFirstChild("Head") and player ~= LocalPlayer then
            pcall(function()
               player.Character.Head.Size = Vector3.new(Value, Value, Value)
               player.Character.Head.CanCollide = false
            end)
         end
      end
   end,
})
