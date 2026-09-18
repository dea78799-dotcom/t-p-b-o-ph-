local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Tập Béo Phì",
   LoadingTitle = "Đang tải script...",
   LoadingSubtitle = "AI HACK",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "TapBeoPhiHub",
      FileName = "Settings"
   },
   Discord = { Enabled = false },
   KeySystem = false
})

-- Services & Remote References
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remote = ReplicatedStorage:WaitForChild("Remote")
local Event = Remote:WaitForChild("Event")
local Function = Remote:WaitForChild("Function")

local EatEvent = Event:WaitForChild("Eat")
local PlayerIsEat = EatEvent:WaitForChild("PlayerIsEat")
local PlayerTryClickRE = EatEvent:WaitForChild("PlayerTryClickRE")

-- Variables
local autoTrainEnabled = false
local autoClaimOfflineEnabled = false
local autoRebirthEnabled = false
local speedEnabled = false
local jumpEnabled = false
local speedValue = 16
local jumpValue = 50

-- Hàm hiển thị thông báo tiện lợi
local function notify(title, content)
   Rayfield:Notify({
      Title = title,
      Content = content,
      Duration = 3,
      Image = 4483362458,
   })
end

---------------------------------------------------------
-- TAB FARM
---------------------------------------------------------
local FarmTab = Window:CreateTab("Farm", 4483362458)

FarmTab:CreateButton({
   Name = "Trang bị thức ăn",
   Callback = function()
      PlayerIsEat:FireServer()
      notify("Farm", "Đã gửi yêu cầu trang bị thức ăn!")
   end,
})

FarmTab:CreateToggle({
   Name = "Tự động tập",
   CurrentValue = false,
   Flag = "AutoTrainToggle",
   Callback = function(Value)
      autoTrainEnabled = Value
      if autoTrainEnabled then
         notify("Tự động tập", "Trạng thái: BẬT (20 lần/giây)")
         PlayerIsEat:FireServer()
         
         task.spawn(function()
            while autoTrainEnabled do
               local args = { true }
               PlayerTryClickRE:FireServer(unpack(args))
               task.wait(1 / 20)
            end
         end)
      else
         notify("Tự động tập", "Trạng thái: TẮT")
      end
   end,
})

FarmTab:CreateToggle({
   Name = "Tự động nhận chất béo khi online",
   CurrentValue = false,
   Flag = "AutoClaimOfflineToggle",
   Callback = function(Value)
      autoClaimOfflineEnabled = Value
      if autoClaimOfflineEnabled then
         notify("Tự nhận chất béo", "Trạng thái: BẬT (Mỗi 10 giây)")
         task.spawn(function()
            while autoClaimOfflineEnabled do
               Event:WaitForChild("Offline"):WaitForChild("ClaimOfflineReward"):FireServer()
               task.wait(10)
            end
         end)
      else
         notify("Tự nhận chất béo", "Trạng thái: TẮT")
      end
   end,
})

FarmTab:CreateToggle({
   Name = "Tự động tái sinh",
   CurrentValue = false,
   Flag = "AutoRebirthToggle",
   Callback = function(Value)
      autoRebirthEnabled = Value
      if autoRebirthEnabled then
         notify("Tự động tái sinh", "Trạng thái: BẬT (Mỗi 10 giây)")
         task.spawn(function()
            while autoRebirthEnabled do
               Event:WaitForChild("Rebirth"):WaitForChild("TryRebirth"):FireServer()
               task.wait(10)
            end
         end)
      else
         notify("Tự động tái sinh", "Trạng thái: TẮT")
      end
   end,
})

FarmTab:CreateButton({
   Name = "Tái sinh ngay bây giờ",
   Callback = function()
      Event:WaitForChild("Rebirth"):WaitForChild("TryRebirth"):FireServer()
      notify("Tái sinh", "Đã thực hiện tái sinh ngay!")
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
      notify("Pet", "Đã gỡ tất cả pet đang trang bị!")
   end,
})

PetTab:CreateButton({
   Name = "Random trứng 1",
   Callback = function()
      local args = { "Egg1", 1 }
      Function:WaitForChild("Luck"):WaitForChild("[C-S]DoLuck"):InvokeServer(unpack(args))
      notify("Pet", "Đã thực hiện mở Trứng 1!")
   end,
})

---------------------------------------------------------
-- TAB THÔNG TIN NGƯỜI CHƠI
---------------------------------------------------------
local PlayerTab = Window:CreateTab("Thông Tin Người Chơi", 4483362458)

PlayerTab:CreateSlider({
   Name = "Tốc độ",
   Range = {1, 500},
   Increment = 1,
   CurrentValue = 16,
   Flag = "SpeedSlider",
   Callback = function(Value)
      speedValue = Value
   end,
})

PlayerTab:CreateToggle({
   Name = "Bật tốc độ",
   CurrentValue = false,
   Flag = "SpeedToggle",
   Callback = function(Value)
      speedEnabled = Value
      if speedEnabled then
         notify("Tốc độ", "Đã BẬT tốc độ chạy: " .. tostring(speedValue))
         task.spawn(function()
            while speedEnabled do
               if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                  LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = speedValue
               end
               task.wait(0.1)
            end
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
               LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
            end
         end)
      else
         notify("Tốc độ", "Đã TẮT tốc độ chạy (Trở về mặc định)")
      end
   end,
})

PlayerTab:CreateSlider({
   Name = "Nhảy cao",
   Range = {1, 500},
   Increment = 1,
   CurrentValue = 50,
   Flag = "JumpSlider",
   Callback = function(Value)
      jumpValue = Value
   end,
})

PlayerTab:CreateToggle({
   Name = "Bật nhảy cao",
   CurrentValue = false,
   Flag = "JumpToggle",
   Callback = function(Value)
      jumpEnabled = Value
      if jumpEnabled then
         notify("Nhảy cao", "Đã BẬT độ cao nhảy: " .. tostring(jumpValue))
         task.spawn(function()
            while jumpEnabled do
               if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                  local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                  hum.UseJumpPower = true
                  hum.JumpPower = jumpValue
               end
               task.wait(0.1)
            end
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
               LocalPlayer.Character:FindFirstChildOfClass("Humanoid").JumpPower = 50
            end
         end)
      else
         notify("Nhảy cao", "Đã TẮT độ cao nhảy (Trở về mặc định)")
      end
   end,
})

PlayerTab:CreateButton({
   Name = "Ragdoll (Nằm)",
   Callback = function()
      Event:WaitForChild("Race"):WaitForChild("Ragdoll"):FireServer()
      notify("Trạng thái", "Đã bật trạng thái Ragdoll (Nằm)")
   end,
})

PlayerTab:CreateButton({
   Name = "Tắt Ragdoll (Đứng dậy)",
   Callback = function()
      Event:WaitForChild("Race"):WaitForChild("UnRagdoll"):FireServer()
      notify("Trạng thái", "Đã tắt trạng thái Ragdoll (Đứng dậy)")
   end,
})

-- Tự động tải lại cài đặt đã lưu
Rayfield:LoadConfiguration()
