-- ========================================================
--         SCRIPT ZBOR COMPLET (PC + MOBIL) – FINAL
-- ========================================================

local UserInputService = game:GetService("UserInputService")
local ContextActionService = game:GetService("ContextActionService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Setări
local isFlying = false
local flySpeed = 100

-- Fizică
local bodyVelocity, bodyGyro

-- Input mobil
local moveUp = false
local moveDown = false
local moveForward = false
local moveBackward = false

-- Acțiuni
local ACTION_TOGGLE_FLY = "ToggleFly"
local ACTION_FLY_UP = "FlyUp"
local ACTION_FLY_DOWN = "FlyDown"
local ACTION_FLY_FORWARD = "FlyForward"
local ACTION_FLY_BACKWARD = "FlyBackward"

-- 🧠 Actualizare zbor la fiecare frame
local function onHeartbeat()
	if isFlying and bodyVelocity and bodyGyro then
		local camera = workspace.CurrentCamera
		local moveDir = Vector3.zero

		-- Sus / jos (PC + Mobil)
		if UserInputService:IsKeyDown(Enum.KeyCode.Space) or moveUp then
			moveDir += Vector3.new(0, 1, 0)
		end
		if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) or UserInputService:IsKeyDown(Enum.KeyCode.RightShift) or moveDown then
			moveDir -= Vector3.new(0, 1, 0)
		end

		-- Față / spate (PC + Mobil)
		if UserInputService:IsKeyDown(Enum.KeyCode.W) or moveForward then
			moveDir += camera.CFrame.LookVector
		end
		if UserInputService:IsKeyDown(Enum.KeyCode.S) or moveBackward then
			moveDir -= camera.CFrame.LookVector
		end

		-- Aplicăm viteza
		if moveDir.Magnitude > 0 then
			bodyVelocity.Velocity = moveDir.Unit * flySpeed
		else
			bodyVelocity.Velocity = Vector3.zero
		end

		-- Orientarea jucătorului spre camera
		bodyGyro.CFrame = CFrame.new(rootPart.Position, rootPart.Position + camera.CFrame.LookVector)
	end
end

-- 🕹️ Pornire / Oprire zbor
local function toggleFly(actionName, inputState, inputObject)
	if inputState ~= Enum.UserInputState.Begin then return end

	isFlying = not isFlying

	if isFlying then
		bodyVelocity = Instance.new("BodyVelocity")
		bodyVelocity.Velocity = Vector3.zero
		bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
		bodyVelocity.Parent = rootPart

		bodyGyro = Instance.new("BodyGyro")
		bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
		bodyGyro.CFrame = rootPart.CFrame
		bodyGyro.Parent = rootPart

		-- 🔘 Activează butoane mobile
		ContextActionService:BindAction(ACTION_FLY_UP, function(_, state)
			moveUp = (state == Enum.UserInputState.Begin)
			return Enum.ContextActionResult.Sink
		end, true)
		ContextActionService:SetTitle(ACTION_FLY_UP, "Sus")
		ContextActionService:SetPosition(ACTION_FLY_UP, UDim2.new(1, -160, 1, -220))

		ContextActionService:BindAction(ACTION_FLY_DOWN, function(_, state)
			moveDown = (state == Enum.UserInputState.Begin)
			return Enum.ContextActionResult.Sink
		end, true)
		ContextActionService:SetTitle(ACTION_FLY_DOWN, "Jos")
		ContextActionService:SetPosition(ACTION_FLY_DOWN, UDim2.new(1, -160, 1, -160))

		ContextActionService:BindAction(ACTION_FLY_FORWARD, function(_, state)
			moveForward = (state == Enum.UserInputState.Begin)
			return Enum.ContextActionResult.Sink
		end, true)
		ContextActionService:SetTitle(ACTION_FLY_FORWARD, "Înainte")
		ContextActionService:SetPosition(ACTION_FLY_FORWARD, UDim2.new(1, -260, 1, -220))

		ContextActionService:BindAction(ACTION_FLY_BACKWARD, function(_, state)
			moveBackward = (state == Enum.UserInputState.Begin)
			return Enum.ContextActionResult.Sink
		end, true)
		ContextActionService:SetTitle(ACTION_FLY_BACKWARD, "Înapoi")
		ContextActionService:SetPosition(ACTION_FLY_BACKWARD, UDim2.new(1, -260, 1, -160))
	else
		-- Dezactivăm fizica
		if bodyVelocity then bodyVelocity:Destroy() end
		if bodyGyro then bodyGyro:Destroy() end
		bodyVelocity = nil
		bodyGyro = nil

		-- Dezactivăm butoanele mobile
		ContextActionService:UnbindAction(ACTION_FLY_UP)
		ContextActionService:UnbindAction(ACTION_FLY_DOWN)
		ContextActionService:UnbindAction(ACTION_FLY_FORWARD)
		ContextActionService:UnbindAction(ACTION_FLY_BACKWARD)

		moveUp, moveDown, moveForward, moveBackward = false, false, false, false
	end

	return Enum.ContextActionResult.Sink
end

-- 🔘 Buton principal zbor (PC: F | Mobil: pe ecran)
ContextActionService:BindAction(ACTION_TOGGLE_FLY, toggleFly, true, Enum.KeyCode.F)
ContextActionService:SetTitle(ACTION_TOGGLE_FLY, "Zbor")
ContextActionService:SetPosition(ACTION_TOGGLE_FLY, UDim2.new(1, -80, 0, 10))

-- ⏱️ Pornim zborul în fiecare frame
RunService.Heartbeat:Connect(onHeartbeat)

print("✅ Script de zbor încărcat (PC + Mobil).")
