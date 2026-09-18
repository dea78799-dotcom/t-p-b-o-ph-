local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Tập Béo Phì",
   LoadingTitle = "Đang tải script...",
   LoadingSubtitle = "Tập Béo Phì Hub",
   ConfigurationSaving = {
      Enabled = false
   },
   Discord = {
      Enabled = false
   },
   KeySystem = false
})

-- Services & Remote References
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Remote = ReplicatedStorage:WaitForChild("Remote")
local Event = Remote:WaitForChild("Event")
local Function = Remote:WaitForChild("Function")

local EatEvent = Event:WaitForChild("Eat")
local PlayerIsEat = EatEvent:WaitForChild("PlayerIsEat")
local PlayerTryClickRE = EatEvent:WaitForChild("PlayerTryClickRE")

-- Variables
local autoTrainEnabled = false

---------------------------------------------------------
-- TAB FARM
---------------------------------------------------------
local FarmTab = Window:CreateTab("Farm", 4483362458)

FarmTab:CreateButton({
   Name = "Trang bị thức ăn",
   Callback = function()
      PlayerIsEat:FireServer()
   end,
})

FarmTab:CreateToggle({
   Name = "Tự động tập",
   CurrentValue = false,
   Flag = "AutoTrainToggle",
   Callback = function(Value)
      autoTrainEnabled = Value
      
      if autoTrainEnabled then
         PlayerIsEat:FireServer()
         
         task.spawn(function()
            while autoTrainEnabled do
               local args = { true }
               PlayerTryClickRE:FireServer(unpack(args))
               task.wait(1 / 50)
            end
         end)
      end
   end,
})

---------------------------------------------------------
-- TAB PET
---------------------------------------------------------
local PetTab = Window:CreateTab("Pet", 4483362458)

PetTab:CreateButton({
   Name = "Gỡ trang bị pet",
   Callback = function()
      Event:WaitForChild("Pet"):WaitForChild("UnEquipAll"):FireServer()
   end,
})

PetTab:CreateButton({
   Name = "Random trứng 1",
   Callback = function()
      local args = {
         "Egg1",
         1
      }
      Function:WaitForChild("Luck"):WaitForChild("[C-S]DoLuck"):InvokeServer(unpack(args))
   end,
})
