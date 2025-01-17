local function checkLine(s)
	local a = 0
	for i in string.gmatch(s, "[^\n]+") do  
		a = a+1
	end
	return a
end
local start = 1
local function addspace(a)
	local st = ""
	for i = 0,a*2 do
		st = st.." "
	end
	return st
end
local function usd(v,i)
	local turn

	if typeof(v) == "Axes" then
		turn = "Axes.new("..tostring(v)..")"
	elseif typeof(v) == "BrickColor" then
		turn = "BrickColor.new("..tostring(v)..")"
	elseif typeof(v) == "CFrame" then
		turn = "CFrame.new("..tostring(v)..")"
	elseif typeof(v) == "Vector3" then
		turn = "Vector3.new("..tostring(v)..")"
	elseif typeof(v) == "Color3" then
		if v.r<=1 and v.g<= 1 and v.b<= 1 and v.r >= 0 and v.g >= 0 and v.b >= 0 then
			local R = v.r * 255
			local G = v.g * 255
			local B = v.b * 255
			turn = 'Color3.fromRGB('..R..", "..G..", "..B..")"..'    --Color3.new('..tostring(v)..")"
		else
			turn = "Color Error"
		end
	elseif typeof(v) == "Instance" then
		turn = "game."..tostring(v:GetFullName())
	elseif typeof(v) == "Enum" then
		turn = tostring(v).."   -- Enum"
	else
		turn = tostring(typeof(v))..".new("..tostring(v)..")"
	end
	if i then
		if type(i) == "number" then 
			return '['..tostring(i)..']'.. " = "..tostring(turn)
		end
		return '["'..tostring(i)..'"]'.. " = "..tostring(turn)
	end
	return tostring(turn)
end

local convert
local startSpace = 1
function convert(v,i,a)
	if not a then
		a = startSpace    
	end
	if type(v) == "string" then
		if checkLine(v) >= 2 then
			if type(i) == "number" then 
				return '['..tostring(i)..']'.. " = "..'[['..tostring(v)..']]'
			end
			return '["'..tostring(i)..'"]'.. " = "..'[['..tostring(v)..']]'
		else
			if type(i) == "number" then 
				return '['..tostring(i)..']'.. " = "..'"'..tostring(v)..'"'
			end
			return '["'..tostring(i)..'"]'.. " = "..'"'..tostring(v)..'"'
		end
	elseif type(v) == "number" then
		if type(i) == "number" then 
			return '['..tostring(i)..']'.. " = "..tostring(v)
		end
		return '["'..tostring(i)..'"]'.. " = "..tostring(v)
	elseif type(v) == "boolean" then
		if type(i) == "number" then 
			return '['..tostring(i)..']'.. " = "..tostring(v)
		end
		return '["'..tostring(i)..'"]'.. " = "..tostring(v)
	elseif type(v) == "nil" then
		if type(i) == "number" then 
			return '['..tostring(i)..']'.. " = "..tostring(v)
		end
		return '["'..tostring(i)..'"]'.. " = nil"
	elseif type(v) == "function" then
		local Name = ""
		if debug.getinfo(v).name == "" then
			Name = tostring(v)
		else
			Name = debug.getinfo(v).name
		end
		if type(i) == "number" then 
			return '['..tostring(i)..']'.. " = "..'function()end  '..'--'.."[["..tostring(Name).."]]"
		end
		return '["'..tostring(i)..'"]'.. " = "..'function()end  '..'--'.."[["..tostring(Name).."]]"
	elseif type(v) == "userdata" or type(v) == "vector" then
		return usd(v,i)
	elseif type(v) == "table" then
		if i~= nil then
			if type(i) == "number" then
				stt = '['..tostring(i)..']'.." = "..'{'
			else
				stt = '["'..tostring(i)..'"]'.." = "..'{'
			end

		else
			stt = '{'
		end
		local count_table = 0
		for i,v in pairs(v) do
			count_table = count_table+1
			if count_table == 1 then
				stt = stt .. "\n" .. tostring(addspace(a)) .. tostring(convert(v, i, a + 1))
			else    
				stt = stt .. ",\n" .. tostring(addspace(a)) .. tostring(convert(v, i, a + 1))
			end
		end
		return stt.."\n}"
	elseif type(v) == "thread" then
		if type(i) == "number" then 
			return '['..tostring(i)..']'.. " = "..tostring(v)
		end
		return '["'..tostring(i)..'"]'.. " = "..tostring(v)
	end
end


local Fluent = loadstring(game:HttpGet("https://raw.githubusercontent.com/Namoqp1/dump/refs/heads/main/them"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
	Title = "Fluent " .. Fluent.Version,
	SubTitle = "by dawid",
	TabWidth = 160,
	Size = UDim2.fromOffset(580, 460),
	Acrylic = true,
	Theme = "Dark",
	MinimizeKey = Enum.KeyCode.LeftControl 
})

local Tabs = {
	Main = Window:AddTab({ Title = "Main", Icon = "" }),
	Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

local nameconfig

local RecordMacroTable = {}

local path = "macro.txt"


local basetime = 0
spawn(function()
	while true do 
		local realtime_wait = wait()
		if game:GetService("Players").LocalPlayer.PlayerGui.VoteStart.Enabled == false then 
			basetime = basetime + realtime_wait
		else
			basetime = 0
		end
	end
end)


local xd = nil

local toggleRecord = Tabs.Main:AddToggle("toggleRecord", {Title = "Start Record", Default = xd })
toggleRecord:OnChanged(function(record)
	xd = record
	if xd then 
		RecordMacroTable = {}
	end
end)


local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)

mt.__namecall = function(self, ...)
	if not checkcaller() then 
		local args = {...}
		if xd then
			if self.Name == "spawn_unit" then
				task.spawn(function()
					if args[2] then
						table.insert(RecordMacroTable, {
							['time'] = basetime,
							['type'] = "Place",
							['data'] = {
								['unit'] = tostring(args[1]),
								['position'] = args[2]
							}
						})
						writefile(path, convert(RecordMacroTable))
					end
				end)
			elseif self.Name == "upgrade_unit_ingame" then
				task.spawn(function()
					table.insert(RecordMacroTable, {
						['time'] = basetime,
						['type'] = "Upgrade",
						['data'] = {
							['position'] = args[1].HumanoidRootPart.CFrame
						}
					})
					writefile(path, convert(RecordMacroTable))
				end)
			elseif self.Name == "sell_unit_ingame" then
				task.spawn(function()
					if args[1] ~= "RemoteFunction" then
						table.insert(RecordMacroTable, {
							['time'] = basetime,
							['type'] = "Sell",
							['data'] = {
								['position'] = args[1].HumanoidRootPart.CFrame
							}
						})
						writefile(path, convert(RecordMacroTable))
					end
				end)
			end
		end
	end
	return old(self, ...)
end



local togglePlay = Tabs.Main:AddToggle("togglePlay", {Title = "Play Macro", Default = _G.Play })
togglePlay:OnChanged(function(play)
	_G.Play = play
	if _G.Play then 
		local datamacro = readfile(path)
		local real = loadstring("return "..datamacro)()
		if #real > 0 and real[#real].time > basetime then
			for i,v in pairs(real) do 
				if _G.Play then
					repeat wait() until basetime >= v.time
					if v["type"] == "Place" then
						local args = {
							[1] = v.data.unit,
							[2] = v.data.position
						}

						game:GetService("ReplicatedStorage"):WaitForChild("endpoints"):WaitForChild("client_to_server"):WaitForChild("spawn_unit"):InvokeServer(unpack(args))
					elseif v["type"] == "Upgrade" then
						
						local current = 1

						local current_instance = nil
						
						for _,unit in pairs(workspace:WaitForChild("_UNITS"):GetChildren()) do
							local dis = (v.data.position.Position-unit.HumanoidRootPart.Position).Magnitude
							if dis < current then
								current = dis
								current_instance = unit.Name
							end
							if current_instance then
								print(current_instance)
								local args = {
									[1] = workspace:WaitForChild("_UNITS"):WaitForChild(current_instance)
								}

								game:GetService("ReplicatedStorage"):WaitForChild("endpoints"):WaitForChild("client_to_server"):WaitForChild("upgrade_unit_ingame"):InvokeServer(unpack(args))
							end
						end
					elseif v["type"] == "Sell" then
						
						local current = 1

						local current_instance = nil
						
						for  _,unit in pairs(workspace:WaitForChild("_UNITS"):GetChildren()) do
							local dis = (v.data.position.Position-unit.HumanoidRootPart.Position).Magnitude
							print(dis)
							if dis < current then
								current = dis
								current_instance = unit.Name
							end
							if current_instance then
								print(current_instance)
								local args = {
									[1] = workspace:WaitForChild("_UNITS"):WaitForChild(current_instance)
								}

								game:GetService("ReplicatedStorage"):WaitForChild("endpoints"):WaitForChild("client_to_server"):WaitForChild("sell_unit_ingame"):InvokeServer(unpack(args))
							end
						end
					end
				end
			end
		end
	end
end)


