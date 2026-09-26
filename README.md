# Bezier-Noise
Bezier Noise is a new type of noise algorithm I made, to explain shorly how it works, connect a bunch of bezier curves together, and every bezier curve is connected by Anchor points and control points with random heights.


##How it works?
The way the algorithm works is pretty simple, first, just create points, you can space them horizontally however you want, but at first I spaced them evenly so each point has an equal distance from the previous point and the next point.

Second step is to just apply a bezier curve for every 3 points, a bezier curve starts from the last point of the previous bezier curve, except for the first one, then you have a control point and the last point, all of which have random heights.

If you had done it like this, there would be most likely alot of sharp turns, to fix this we need the control point of every curve to have the same slope as the line that the last point of the previous bezier curve and the control point of the previous bezier curve makes. After this change we can draw bezier curves and there won't be any sharp turns.

```lua
--This program was made by pirssy_slipped
--Bezier noise was invented by pirssy_slipped
--\\Services
local PlayersService = game:GetService("Players")
--\\Variables

local player = PlayersService.LocalPlayer
local mouse = player:GetMouse()

local Board = script.Parent.Board
local Paint = script.Parent.Paint

local origin = Vector2.new(0,0)

local ContorlPoints = {}
local PaintPoints = {}
local PaintAmount = 50

local controlPoint = Board.ControlPoint
local anchor = Board.Anchor
local anchor2 = Board.Anchor2
local wait = false


local drag = false
--\\Program

local function CalculateSlope(v1 : Vector2,v2 : Vector2)
	local slope = v1.Y - v2.Y/v1.X - v2.X
	return slope
end

--[[local function draw_bezier()
	for t = 0, 1, 1/PaintAmount do
		local frame
		local line1 = anchor.Position:Lerp(controlPoint.Position, t)
		local line2 = controlPoint.Position:Lerp(anchor2.Position, t)

		local bezier = line1:Lerp(line2, t)
		print(t*PaintAmount)
		local paint = PaintPoints[math.floor(t * 100)]
		paint.Parent = Board
		paint.Visible = true
		paint.Position = UDim2.new(0, bezier.X,0, bezier.Y)
	end
end]]

local function draw_bezier(p1 : Vector2,p2 : Vector2,p3 : Vector2)
	for t = 0, 1, 1/PaintAmount do
		if p3 then
			local paint = Paint:Clone()
			local line1 = p1:Lerp(p2,t)
			local line2 = p2:Lerp(p3,t)

			local bezier = line1:Lerp(line2, t)
			--local paint = PaintPoints[math.floor(t * 100)]
			paint.Parent = Board
			paint.Visible = true
			paint.Position = UDim2.new(0, bezier.X,0, bezier.Y)
			if wait then
				task.wait()
			end
		end
	end
end

local function CreatePaint(Amount)
	for i = 0, Amount do
		local paint = Paint:Clone()
		PaintPoints[i] = paint
		paint.Visible = true
	end
end

CreatePaint(PaintAmount)

local function Update_drag(point : Frame)
	if drag == true then
		while true do
			local xDISTANCE = mouse.X - Board.AbsolutePosition.X
			local yDISTANCE = mouse.Y - Board.AbsolutePosition.Y
			point.Position = UDim2.new(math.clamp(xDISTANCE/Board.AbsoluteSize.X,0,1),0,math.clamp(yDISTANCE/Board.AbsoluteSize.Y,0,1),0)
			task.wait()
		end
	end
end

local function CreateControlPoints(Amount)
	local middle = Board.AbsoluteSize.Y/2
	for i = 1, Amount do
		local point = Vector2.new((Board.AbsoluteSize.X/Amount) * i + math.random(5,20), math.random(middle-50,middle+50))
		local knot = math.random(1,100)
		if i > 3 and (i - 1) % 2 ~= 0 then -- Continuity
			local previousPoint = ContorlPoints[i-1]
			local previousPoint2 = ContorlPoints[i-2]
			local slope = (previousPoint - previousPoint2)
			local desiredDirecetion = previousPoint + slope/2
			point = Vector2.new(desiredDirecetion.X,math.clamp(desiredDirecetion.Y, 0,Board.AbsoluteSize.Y))
		end
		local clone = controlPoint:Clone()
		clone.Position = UDim2.new(0,point.X,0,point.Y)
		clone.Visible = true
		ContorlPoints[i] = point

	end
end

task.spawn(function()
	controlPoint.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			drag = true
			Update_drag(controlPoint)
		end
	end)

	controlPoint.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			drag = false
		end
	end)


end)


local function generate(amount)
	for i, v in pairs(Board:GetChildren()) do
		if v:IsA("Frame") then
			v:Destroy()
		end
	end
	table.clear(ContorlPoints)
	CreateControlPoints(amount)
	for i, v in pairs(ContorlPoints) do
		local frame = Instance.new("Frame")
		frame.Size = UDim2.new(0,5,0,5)
		frame.Position = UDim2.new(0,ContorlPoints[i].X,0,ContorlPoints[i].Y)
		frame.Parent = Board
		frame.Visible = true
		
		frame.ZIndex = 2
		frame.BackgroundColor3 = Color3.new(255,255,255)
		if i > 3 and (i - 1) % 2 ~= 0 then
			frame.BackgroundColor3 = Color3.new(1, 0, 0.0156863)
		end
		task.wait(.5)
	end
	task.wait(2)
	for i = 1, #ContorlPoints, 2 do
		local point1 = ContorlPoints[i]
		local point2 = ContorlPoints[i+1] or nil
		local point3 = ContorlPoints[i+2] or nil

		draw_bezier(point1,point2,point3)
	end
end

script.Parent.Generate.MouseButton1Click:Connect(function()
	generate(20)
end)

script.Parent.Animation.MouseButton1Click:Connect(function()
	if wait == false then
		wait = true
	else
		wait = false
	end
end)```
