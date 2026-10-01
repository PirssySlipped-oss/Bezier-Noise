# Bézier-Noise
Bezier Noise is a new type of noise algorithm I made, to explain shortly how it works, connect a bunch of bezier curves together, and every bezier curve is connected by Anchor points and control points with random heights.

## What is a Bézier curve?
Bézier curve is a curve that was invented by Pierre Bézier and Paul de Casteljau. To explain briefly, it is cruve a that is defined by two anchor points and one control point (In the following image, the three points make up a triangle, the control point is the point at the top, the points that make the base of the triangle, points 2 and 3, are the anchor points), now imagine sliding another points across the the first anchor point and the control point, and at the same time sliding another point from the control point to the last anchor point, the positions of the sliding points is the defined by the variable t, which goes from 0 to 1, now imagine drawing a line from the two sliding points, and on that line have another point that is sliding from the first sliding point to the other. The position of that point is defined by the same variable t, and as the point slides it traces a prefect smooth curve.

I took the following image from: https://javascript.info/bezier-curve, which explaines perfectly how it works, visit it if you are interested!

<img width="809" height="404" alt="image" src="https://github.com/user-attachments/assets/ced6c78f-b516-4202-8912-3fb44f500f54" />



## How it works?

The way the algorithm works is pretty simple, first, just create points, you can space them horizontally however you want, but at first I spaced them evenly so each point has an equal distance from the previous point and the next point.

Second step is to just apply a bezier curve for every 3 points, a bezier curve starts from the last point of the previous bezier curve, except for the first one, then you have a control point and the last point, all of which have random heights. Here is how it looks like:

<img width="611" height="187" alt="image" src="https://github.com/user-attachments/assets/4fb02dd4-414b-480c-b9c4-7215b479d2fd" />


Now, If you had done it like this, there would be most likely alot of sharp turns, to fix this we need the control point of every curve to have the same slope as the line that the last point of the previous bezier curve and the control point of the previous bezier curve makes. After this change we can draw bezier curves and there won't be any sharp turns.

The following program I am going to share with you is the code for drawing the graph in roblox studio. I don't expect you to understand it if you don't know the programming language of roblox, Luau, but i recommend you read the lines that describe the function of plotting the anchor points and control points, which is called "CreateControlPoints(Amount)", and I also recommend to read the function that actually draws the bezier curve, which is called "draw_bezier(p1 : Vector2,p2 : Vector2,p3 : Vector2)"

```lua
--This program was made by pirssy_slipped
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

-- The function takes three points, lerps between the first point and second point, then lerps between the second point and last point, and then for every lerp iteration, it lerps between the points that are "sliding" on the lines between the first point and the second point, and the line between second point and the last point
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
		if i > 3 and (i - 1) % 2 ~= 0 then -- Continuity solution:
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
end)
```
This is a demonstration to how it works in roblox studio:

<img width="800" height="600" alt="BezierCurveDemo-ezgif com-video-to-gif-converter" src="https://github.com/user-attachments/assets/dbabea19-6ff0-4d7b-82f7-bef59355f7b2" />

The red dots you see are control points that have the same slope as the previous points. If you look closely, they pretty much sit on the same line

<img width="435" height="215" alt="image" src="https://github.com/user-attachments/assets/5d6d89e5-8994-4685-a98a-b0282e30722f" />

Here are more exmaples for Bezier Noise

<img width="606" height="243" alt="image" src="https://github.com/user-attachments/assets/c4e007d4-e84c-416d-a6d1-56e5df43fb62" />

<img width="611" height="238" alt="image" src="https://github.com/user-attachments/assets/32232071-05f4-4fc4-a774-73b8e2f2247f" />

<img width="608" height="239" alt="image" src="https://github.com/user-attachments/assets/98966d8f-a13c-4f75-adfd-b4a0714c13cf" />

<img width="608" height="247" alt="image" src="https://github.com/user-attachments/assets/f44cf46e-6c42-48e5-b22b-d0cb058237e5" />

## Credits to Paul de Casteljau, Pierre Bézier and Pirssy Slipped
