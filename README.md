local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera
local AimbotEnabled = false
local UIS = game:GetService("UserInputService")
local Mouse = LocalPlayer:GetMouse()
local SelectedTarget = nil -- Guarda o alvo que estamos mirando

-- Nome exato da arma que queremos ativar o aimbot
local ArmaPermitida = "Arisaka 7.7mm"

-- Criar botão de ativação/desativação
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local ToggleButton = Instance.new("TextButton", ScreenGui)

ToggleButton.Size = UDim2.new(0, 200, 0, 50)
ToggleButton.Position = UDim2.new(0.1, 0, 0.1, 0)
ToggleButton.Text = "Aimbot OFF"
ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.BackgroundTransparency = 0.2  

ToggleButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    if AimbotEnabled then
        ToggleButton.Text = "Aimbot ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    else
        ToggleButton.Text = "Aimbot OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        SelectedTarget = nil -- Resetar alvo se o aimbot for desligado
    end
end)

-- Verifica se o jogador está segurando a arma certa
local function TemArmaCerta()
    local Character = LocalPlayer.Character
    if Character and Character:FindFirstChildOfClass("Tool") then
        local Arma = Character:FindFirstChildOfClass("Tool")
        return Arma.Name == ArmaPermitida
    end
    return false
end

-- Função para encontrar o inimigo que já estamos mirando
local function GetCurrentTarget()
    local ShortestDistance = math.huge
    local MelhorAlvo = nil

    for _, Player in pairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") then
            if Player.TeamColor == BrickColor.new("Bright red") then
                local Head = Player.Character.Head
                local DistanceFromPlayer = (Head.Position - LocalPlayer.Character.Head.Position).Magnitude
                
                -- Definir a meia distância (máximo de 150 studs)
                if DistanceFromPlayer < 150 then
                    local EnemyPos, OnScreen = Camera:WorldToViewportPoint(Head.Position)
                    if OnScreen then
                        local DistanceToCenter = (Vector2.new(EnemyPos.X, EnemyPos.Y) - UserInputService:GetMouseLocation()).Magnitude
                        if DistanceToCenter < ShortestDistance and DistanceToCenter < 100 then
                            ShortestDistance = DistanceToCenter
                            MelhorAlvo = Head
                        end
                    end
                end
            end
        end
    end
    return MelhorAlvo
end

-- Função de mira suavizada para perseguir o inimigo
local function SmoothAim(Target, Speed)
    if Target then
        local TargetPos = Target.Position
        local CurrentCFrame = Camera.CFrame
        local NewDirection = (TargetPos - CurrentCFrame.Position).unit

        -- Suaviza a transição para a mira
        local SmoothedCFrame = CFrame.new(CurrentCFrame.Position, CurrentCFrame.Position + (CurrentCFrame.LookVector:Lerp(NewDirection, Speed)))
        Camera.CFrame = SmoothedCFrame
    end
end

-- Função para aumentar a precisão no tiro para 90% na cabeça
local function AimbotShoot(Target)
    if Target then
        SmoothAim(Target, 0.9) -- Puxa a mira para a cabeça com 90% de precisão
    end
end

-- Ativar mira ao atirar, mas só se estiver com o Arisaka e mirando o mesmo alvo
Mouse.Button1Down:Connect(function()
    if AimbotEnabled and TemArmaCerta() then
        if not SelectedTarget then
            SelectedTarget = GetCurrentTarget() -- Escolher o inimigo que estamos mirando no momento
        end
        if SelectedTarget then
            AimbotShoot(SelectedTarget) -- Puxa 90% para a cabeça do inimigo
        end
    end
end)

-- Função principal do Aimbot (persegue só o inimigo que estamos mirando)
local function Aimbot()
    while true do
        if AimbotEnabled and TemArmaCerta() then
            if SelectedTarget then
                -- Seguir o alvo com 100% de precisão, se já estamos mirando nele
                SmoothAim(SelectedTarget, 1) -- 100% para o inimigo que estamos mirando
            else
                SelectedTarget = GetCurrentTarget() -- Tenta escolher um novo alvo se não tiver um
            end
        else
            SelectedTarget = nil -- Resetar alvo se não estiver com a arma certa
        end
        task.wait(0.01)  
    end
end

-- Iniciar Aimbot
task.spawn(Aimbot)
