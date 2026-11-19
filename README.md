-- Fly GUI Moderno para Roblox (Compatible con Delta Executor)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Configuración del vuelo
local FLY_SPEED = 60
local flying = false
local minimized = false
local moveConnection = nil
local bodyVelocity = nil
local bodyGyro = nil

-- Limpiar GUI previa si se re-ejecuta el script
local parentGui = (gethui and gethui()) or player:WaitForChild("PlayerGui")
if parentGui:FindFirstChild("ModernFlyGUI") then
	parentGui.ModernFlyGUI:Destroy()
end

-- Crear ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ModernFlyGUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = parentGui

-- Frame Principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 260, 0, 155)
mainFrame.Position = UDim2.new(0.5, -130, 0.35, -77)
mainFrame.BackgroundColor3 = Color3.fromRGB(16, 16, 20)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 12)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(185, 28, 46)
mainStroke.Thickness = 1.5
mainStroke.Parent = mainFrame

-- Barra Superior (Arrastrable)
local topBar = Instance.new("Frame")
topBar.Name = "TopBar"
topBar.Size = UDim2.new(1, 0, 0, 38)
topBar.BackgroundColor3 = Color3.fromRGB(24, 24, 30)
topBar.BorderSizePixel = 0
topBar.Parent = mainFrame

local topCorner = Instance.new("UICorner")
topCorner.CornerRadius = UDim.new(0, 12)
topCorner.Parent = topBar

-- Tapar esquinas inferiores de la barra superior
local topFix = Instance.new("Frame")
topFix.Size = UDim2.new(1, 0, 0, 12)
topFix.Position = UDim2.new(0, 0, 1, -12)
topFix.BackgroundColor3 = Color3.fromRGB(24, 24, 30)
topFix.BorderSizePixel = 0
topFix.Parent = topBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -80, 1, 0)
titleLabel.Position = UDim2.new(0, 14, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "FLY CONTROL"
titleLabel.TextColor3 = Color3.fromRGB(245, 245, 250)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 14
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = topBar

-- Botón Minimizar (-)
local minButton = Instance.new("TextButton")
minButton.Name = "MinimizeButton"
minButton.Size = UDim2.new(0, 28, 0, 28)
minButton.Position = UDim2.new(1, -34, 0.5, -14)
minButton.BackgroundColor3 = Color3.fromRGB(38, 38, 48)
minButton.Text = "-"
minButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minButton.Font = Enum.Font.GothamBold
minButton.TextSize = 18
minButton.AutoButtonColor = false
minButton.Parent = topBar

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 8)
minCorner.Parent = minButton

-- Botón flotante para reabrir cuando está minimizado
local reopenButton = Instance.new("TextButton")
reopenButton.Name = "ReopenButton"
reopenButton.Size = UDim2.new(0, 120, 0, 36)
reopenButton.Position = mainFrame.Position
reopenButton.BackgroundColor3 = Color3.fromRGB(185, 28, 46)
reopenButton.Text = "Abrir Fly"
reopenButton.TextColor3 = Color3.fromRGB(255, 255, 255)
reopenButton.Font = Enum.Font.GothamBold
reopenButton.TextSize = 13
reopenButton.Visible = false
reopenButton.Active = true
reopenButton.Parent = screenGui

local reopenCorner = Instance.new("UICorner")
reopenCorner.CornerRadius = UDim.new(0, 10)
reopenCorner.Parent = reopenButton

-- Contenedor de Contenido
local contentFrame = Instance.new("Frame")
contentFrame.Name = "ContentFrame"
contentFrame.Size = UDim2.new(1, -24, 1, -50)
contentFrame.Position = UDim2.new(0, 12, 0, 44)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

-- Indicador de Estado
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 0, 22)
statusLabel.Position = UDim2.new(0, 0, 0, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Estado: DESACTIVADO"
statusLabel.TextColor3 = Color3.fromRGB(220, 70, 85)
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 12
statusLabel.Parent = contentFrame

-- Botón ACTIVAR
local activateBtn = Instance.new("TextButton")
activateBtn.Name = "ActivateButton"
activateBtn.Size = UDim2.new(0.48, 0, 0, 46)
activateBtn.Position = UDim2.new(0, 0, 0, 30)
activateBtn.BackgroundColor3 = Color3.fromRGB(22, 163, 74)
activateBtn.Text = "ACTIVAR"
activateBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
activateBtn.Font = Enum.Font.GothamBold
activateBtn.TextSize = 13
activateBtn.AutoButtonColor = false
activateBtn.Parent = contentFrame

local actCorner = Instance.new("UICorner")
actCorner.CornerRadius = UDim.new(0, 8)
actCorner.Parent = activateBtn

-- Botón DESACTIVAR
local deactivateBtn = Instance.new("TextButton")
deactivateBtn.Name = "DeactivateButton"
deactivateBtn.Size = UDim2.new(0.48, 0, 0, 46)
deactivateBtn.Position = UDim2.new(0.52, 0, 0, 30)
deactivateBtn.BackgroundColor3 = Color3.fromRGB(153, 27, 27)
deactivateBtn.Text = "DESACTIVAR"
deactivateBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
deactivateBtn.Font = Enum.Font.GothamBold
deactivateBtn.TextSize = 13
deactivateBtn.AutoButtonColor = false
deactivateBtn.Parent = contentFrame

local deactCorner = Instance.new("UICorner")
deactCorner.CornerRadius = UDim.new(0, 8)
deactCorner.Parent = deactivateBtn

-- Ayuda rápida de controles
local hintLabel = Instance.new("TextLabel")
hintLabel.Size = UDim2.new(1, 0, 0, 18)
hintLabel.Position = UDim2.new(0, 0, 1, -18)
hintLabel.BackgroundTransparency = 1
hintLabel.Text = "WASD: Mover | Space: Subir | Ctrl: Bajar"
hintLabel.TextColor3 = Color3.fromRGB(150, 150, 165)
hintLabel.Font = Enum.Font.Gotham
hintLabel.TextSize = 11
hintLabel.Parent = contentFrame

-- Función para hacer cualquier Frame arrastrable (Mouse y Touch en móvil)
local function makeDraggable(frame, dragHandle)
	local dragging = false
	local dragInput = nil
	local dragStart = nil
	local startPos = nil

	local function update(input)
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end

	dragHandle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = frame.Position

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	dragHandle.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			update(input)
		end
	end)
end

makeDraggable(mainFrame, topBar)
makeDraggable(reopenButton, reopenButton)

-- Funciones de limpieza y vuelo
local function cleanupFlight()
	if moveConnection then
		moveConnection:Disconnect()
		moveConnection = nil
	end
	if bodyVelocity then
		bodyVelocity:Destroy()
		bodyVelocity = nil
	end
	if bodyGyro then
		bodyGyro:Destroy()
		bodyGyro = nil
	end

	local character = player.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.PlatformStand = false
	end
end

local function setFlyState(enabled)
	flying = enabled
	cleanupFlight()

	if flying then
		statusLabel.Text = "Estado: ACTIVADO"
		statusLabel.TextColor3 = Color3.fromRGB(74, 222, 128)

		local character = player.Character or player.CharacterAdded:Wait()
		local rootPart = character:WaitForChild("HumanoidRootPart")
		local humanoid = character:WaitForChild("Humanoid")

		humanoid.PlatformStand = true

		bodyVelocity = Instance.new("BodyVelocity")
		bodyVelocity.Name = "FlyVelocity"
		bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
		bodyVelocity.Velocity = Vector3.zero
		bodyVelocity.Parent = rootPart

		bodyGyro = Instance.new("BodyGyro")
		bodyGyro.Name = "FlyGyro"
		bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
		bodyGyro.P = 1e4
		bodyGyro.CFrame = rootPart.CFrame
		bodyGyro.Parent = rootPart

		moveConnection = RunService.RenderStepped:Connect(function()
			if not flying or not rootPart or not rootPart.Parent then
				cleanupFlight()
				return
			end

			camera = workspace.CurrentCamera
			local camCFrame = camera.CFrame
			local moveDir = Vector3.zero

			-- Movimiento horizontal y frontal según cámara
			if UserInputService:IsKeyDown(Enum.KeyCode.W) then
				moveDir += camCFrame.LookVector
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.S) then
				moveDir -= camCFrame.LookVector
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.A) then
				moveDir -= camCFrame.RightVector
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.D) then
				moveDir += camCFrame.RightVector
			end

			-- Subir y bajar
			if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
				moveDir += Vector3.new(0, 1, 0)
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.RightControl) then
				moveDir -= Vector3.new(0, 1, 0)
			end

			if moveDir.Magnitude > 0 then
				moveDir = moveDir.Unit
			end

			bodyVelocity.Velocity = moveDir * FLY_SPEED
			bodyGyro.CFrame = CFrame.lookAt(rootPart.Position, rootPart.Position + camCFrame.LookVector)
		end)
	else
		statusLabel.Text = "Estado: DESACTIVADO"
		statusLabel.TextColor3 = Color3.fromRGB(220, 70, 85)
	end
end

-- Conectar botones ACTIVAR / DESACTIVAR
activateBtn.MouseButton1Click:Connect(function()
	setFlyState(true)
end)

deactivateBtn.MouseButton1Click:Connect(function()
	setFlyState(false)
end)

-- Minimizar y reabrir
minButton.MouseButton1Click:Connect(function()
	minimized = true
	reopenButton.Position = mainFrame.Position
	mainFrame.Visible = false
	reopenButton.Visible = true
end)

reopenButton.MouseButton1Click:Connect(function()
	minimized = false
	mainFrame.Position = reopenButton.Position
	reopenButton.Visible = false
	mainFrame.Visible = true
end)

-- Asegurar que al reaparecer el personaje no quede con estado colgado
player.CharacterAdded:Connect(function()
	if flying then
		task.wait(0.4)
		setFlyState(true)
	else
		cleanupFlight()
	end
end)


