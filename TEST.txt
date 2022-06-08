;確認是否使用管理員身份開啟
full_command_line := DllCall("GetCommandLine", "str")
if not (A_IsAdmin or RegExMatch(full_command_line, " /restart(?!\S)"))
{
    try
    {
        if A_IsCompiled
            Run *RunAs "%A_ScriptFullPath%" /restart
        else
            Run *RunAs "%A_AhkPath%" /restart "%A_ScriptFullPath%"
    }
    ExitApp
}

testdownlad

;Fetch版本，確認是否有新版
vr := ComObjCreate("WinHttp.WinHttpRequest.5.1")
vr.Open("GET", "https://pouflut.github.io/BNS/version", true)
vr.Send()
vr.WaitForResponse()
version := vr.ResponseText
iniRead, res, version.ini, version, version
res := res "`n"
if ( version != res ){
	MsgBox, 有新版本
	ExitApp
}

;動態調整視窗名稱
Loop{
	if WinExist( "巨集整合版-"A_index ){
	}
	Else{
		wincnt := A_index
		Break
	}
}

;取得劍靈視窗ID
WinGet, hwnd, IDLast, ahk_class UnrealWindow

#If WinActive( "ahk_id" hwnd )

#Include, <classMemory>
#Include, <memoryWriteFunc>
#include, <func>
#include, <marco>

#SingleInstance off

SetKeyDelay, -1, -1, play
CoordMode, Pixel, Client
CoordMode, Mouse, Client

Global speed_now := 1, susFg := 0, pX, pY, pZ, ColorSetting, SkillFg

iniRead, action, user.ini, hotkey_setting, action
iniRead, mode, user.ini, hotkey_setting, mode
iniRead, susKey, user.ini, hotkey_setting, suspend
iniRead, pauseKey, user.ini, hotkey_setting, pause
iniRead, reloadKey, user.ini, hotkey_setting, reload
iniRead, Macro, user.ini, hotkey_setting, macro
iniRead, getPosKey, user.ini, hotkey_setting, getPosKey
iniRead, speedOn, user.ini, hotkey_setting, speedOn
iniRead, speedOff, user.ini, hotkey_setting, speedOff
iniRead, speed, user.ini, hotkey_setting, speed
iniRead, TPKey, user.ini, hotkey_setting, TPKey
iniRead, TPChecked, user.ini, hotkey_setting, TPChecked
iniRead, HotkeyChecked, user.ini, hotkey_setting, HotkeyChecked
iniRead, address, pos.ini, address, address
address := SubStr( address, 3)

Gui, +AlwaysOnTop
Gui, Font, s10,Microsoft JhengHei
Gui, Color, 0xf0f0f0
Gui, Add, Tab3, x20 y20 w260 h430 +Buttons +0x8, 常用|按鍵|龍脈|取色|

;常用分頁
Gui, Tab, 常用
Gui Add, Text, x40 y68 w80 h40, 巨        集

Switch Macro
{
	Case "none":
		Gui Add, DDL, x115 y65 w120 h40 r5 gChangeMacro vMacroChoice, 無||右鍵連點|咒術-死神|燐劍-風系(簡化)|天道-銀河(簡化)
	Case "TOnly":
		Gui Add, DDL, x115 y65 w120 h40 r5 gChangeMacro vMacroChoice, 無|右鍵連點||咒術-死神|燐劍-風系(簡化)|天道-銀河(簡化)
	Case "Warlock_third":
		Gui Add, DDL, x115 y65 w120 h40 r5 gChangeMacro vMacroChoice, 無|右鍵連點|咒術-死神||燐劍-風系(簡化)|天道-銀河(簡化)
		loadSetting( "skill_setting_warlock_third" )
	Case "Blade_Dancer_wind":
		Gui Add, DDL, x115 y65 w120 h40 r5 gChangeMacro vMacroChoice, 無|右鍵連點|咒術-死神|燐劍-風系(簡化)||天道-銀河(簡化)
	Case "Astromancer_starcaster":
		Gui Add, DDL, x115 y65 w120 h40 r5 gChangeMacro vMacroChoice, 無|右鍵連點|咒術-死神|燐劍-風系(簡化)|天道-銀河(簡化)||
}

Gui Add, Text, x40 y118 w80 h40, 巨集按鍵
Switch action
{
	Case "LButton":
		Gui Add, DDL, x115 y115 w60 h40 r5 gChangeKey vKeyChoice, 左鍵||右鍵|側鍵1|側鍵2|
	Case "RButton":
		Gui Add, DDL, x115 y115 w60 h40 r5 gChangeKey vKeyChoice, 左鍵|右鍵||側鍵1|側鍵2|
	Case "XButton1":
		Gui Add, DDL, x115 y115 w60 h40 r5 gChangeKey vKeyChoice, 左鍵|右鍵|側鍵1||側鍵2|
	Case "XButton2":
		Gui Add, DDL, x115 y115 w60 r5 gChangeKey vKeyChoice, 左鍵|右鍵|側鍵1|側鍵2||
}

If ( HotkeyChecked == 0){
	Gui Add, CheckBox, x190 y113 h30 vHotkeyBtn gChangeHotkeyBtn, 啟用
	GuiControl, Disable, MacroChoice
	GuiControl, Disable, KeyChoice
}
Else{
	Gui Add, CheckBox, x190 y113 h30 checked 1 vHotkeyBtn gChangeHotkeyBtn, 啟用
	Hotkey, %action%, Action, on, T2
}

Gui Add, Text, x40 y168 w80 h40, 模        式
If ( mode == 1 ){
	Gui Add, Radio, x115 y158 h40 checked 1 gModeOne, 按一下
	Gui Add, Radio, x190 y158 h40 gModeTwo, 按住
}
Else{
	Gui Add, Radio, x115 y158 h40 gModeOne, 按一下
	Gui Add, Radio, x190 y158 h40 checked 1 gModeTwo, 按住
}
Gui Add, GroupBox, x40 y210 w185 h150, 加速
Gui Add, Text, x55 y238 w80, 加速倍率
Gui Add, Edit, x120 y235 w90 r1 +Number +Limit2 vSpeedTimes, %speed%
GuiControl, Disable, SpeedTimes
Gui Add, Text, x55 y278 w80, 基        址
Gui Add, Edit, x120 y275 w90 r1 vAddress, %address%
GuiControl, Disable, Address
Gui Add, Button, x50 y315 w100 h30 gSave, 儲存
Gui Add, CheckBox, x165 y315 h30 vEdit_one gChangeEdit_one, 編輯
Gui, Font, s8,Microsoft JhengHei
Gui Add, Text, x90 y400 w120, © 2022 溼答答的喔
Gui, Font, s10,Microsoft JhengHei

;按鍵分頁
Gui, Tab, 按鍵
Gui Add, Text, x50 y68 w80, 開啟加速
Gui Add, Hotkey, x130 y65 w90 vSpeedOn, %speedOn%
GuiControl, Disable, SpeedOn
Gui Add, Text, x50 y118 w80, 關閉加速
Gui Add, Hotkey, x130 y115 w90 vSpeedOff, %speedOff%
GuiControl, Disable, SpeedOff
If ( speedOn == speedOff )
	Hotkey, %speedOn%, Speed, on
Else{
	Hotkey, %speedOn%, SpeedOn, on
	Hotkey, %speedOff%, SpeedOff, on
}
Gui Add, Text, x35 y168 h40, 啟用/禁用按鍵
Gui Add, Hotkey, x130 y165 w90 vSus, %susKey%
Hotkey, %susKey%, Sus, on
GuiControl, Disable, Sus
Gui Add, Text, x50 y218 h40, 強制停止
Gui Add, Hotkey, x130 y215 w90 vPus, %pauseKey%
Hotkey, %pauseKey%, Pus, on
GuiControl, Disable, Pus
Gui Add, Text, x50 y268 h40, 重啟程式
Gui Add, Hotkey, x130 y265 w90 vRe, %reloadKey%
Hotkey, %reloadKey%, Re, on
GuiControl, Disable, Re
Gui Add, Button, x50 y315 w100 h30 gSave, 儲存
Gui Add, CheckBox, x165 y315 h30 vEdit_two gChangeEdit_two, 編輯
Gui, Font, s8,Microsoft JhengHei
Gui Add, Text, x90 y400 w120, © 2022 溼答答的喔
Gui, Font, s10,Microsoft JhengHei

;龍脈分頁
Gui, Tab, 龍脈
Gui Add, GroupBox, x25 y55 w220 h125, 英雄副本
Gui Add, GroupBox, x25 y180 w220 h75, 封魔副本
Gui Add, GroupBox, x25 y255 w220 h50, 十二人本
Gui Add, Radio, x35 y80 vC_E1 gE1, `  流浪船一王
Gui Add, Radio, x35 y105 vC_E2 gE2, `  流浪船二王
Gui Add, Radio, x35 y130 vC_E3 gE3, `  夢幻天樹林
Gui Add, Radio, x35 y155 vC_E4 gE4, `  南天藏寶庫
Gui Add, Radio, x145 y80 vC_E5 gE5, `  彎勾谷
Gui Add, Radio, x145 y105 vC_E6 gE6, `  三途川鬼橋
Gui Add, Radio, x145 y130 vC_E7 gE7, `  喚雷峽谷
Gui Add, Radio, x145 y155 vC_E8 gE8, `  沙暴神殿
Gui Add, Radio, x35 y205 vC_S1 gS1, `封魔-傳承祠堂
Gui Add, Radio, x35 y230 vC_S2 gS2, `封魔-試煉殿堂
Gui Add, Radio, x145 y205 vC_S3 gS3, ` 封魔-火炮蘭
Gui Add, Radio, x145 y230 vC_S4 gS4, ` 封魔-崑崙派
Gui Add, Radio, x35 y280 vC_T1 gT1, ` 安息一王
Gui Add, Radio, x145 y280 vC_T2 gT2, ` 安息二王
Gui Add, GroupBox, x25 y305 w220 h90
Gui Add, Text, x35 y328 w80, 龍脈按鍵
Gui Add, Hotkey, x100 y323 w80 vTPKey, %TPKey%
If ( TPChecked == 0 ){
	Hotkey, %TPKey%, TP, off
	Gui Add, CheckBox, x190 y320 h30 vTPBtn gChangeTPBtn, 啟用
	GuiControl, Disable, C_E1
	GuiControl, Disable, C_E2
	GuiControl, Disable, C_E3
	GuiControl, Disable, C_E4
	GuiControl, Disable, C_E5
	GuiControl, Disable, C_E6
	GuiControl, Disable, C_E7
	GuiControl, Disable, C_E8
	GuiControl, Disable, C_S1
	GuiControl, Disable, C_S2
	GuiControl, Disable, C_S3
	GuiControl, Disable, C_S4
	GuiControl, Disable, C_T1
	GuiControl, Disable, C_T2
}
Else{
	Hotkey, %TPKey%, TP, on
	Gui Add, CheckBox, x190 y320 h30 checked 1 vTPBtn gChangeTPBtn, 啟用
	GuiControl, Enable, C_E1
	GuiControl, Enable, C_E2
	GuiControl, Enable, C_E3
	GuiControl, Enable, C_E4
	GuiControl, Enable, C_E5
	GuiControl, Enable, C_E6
	GuiControl, Enable, C_E7
	GuiControl, Enable, C_E8
	GuiControl, Enable, C_S1
	GuiControl, Enable, C_S2
	GuiControl, Enable, C_S3
	GuiControl, Enable, C_S4
	GuiControl, Enable, C_T1
	GuiControl, Enable, C_T2
}
GuiControl, Disable, TPKey
Gui Add, Button, x50 y355 w100 h30 gSave, 儲存
Gui Add, CheckBox, x165 y355 h30 vEdit_three gChangeEdit_three, 編輯
Gui, Font, s8,Microsoft JhengHei
Gui Add, Text, x90 y400 w120, © 2022 溼答答的喔
Gui, Font, s10,Microsoft JhengHei

;取色分頁
Gui, Tab, 取色
Gui Add, Text, x45 y68 w80 h40, 取色職業
Gui Add, DDL, x125 y65 w100 h40 r5 gSkillContent vColorChoice, 咒術-死神|
Gui Add, Radio, x50 y98 h40 w200 vC_one gColorOne, 技能一
Gui Add, Radio, x50 y138 h40 w200 vC_two gColorTwo, 技能二
Gui Add, Radio, x50 y178 h40 w200 vC_three gColorThree, 技能三
Gui Add, Radio, x50 y218 h40 w200 vC_four gColorFour, 技能四
GuiControl, hide, C_one
GuiControl, hide, C_two
GuiControl, hide, C_three
GuiControl, hide, C_four
GuiControl, Disable, ColorChoice
Gui Add, GroupBox, x25 y305 w220 h90
Gui Add, Text, x35 y328 w80, 取色按鍵
Gui Add, Hotkey, x100 y323 w80 vGetPos, %getPosKey%
Gui Add, CheckBox, x190 y320 h30 vGetBtn gChangeGetBtn, 啟用
GuiControl, Disable, GetPos
Gui Add, Button, x50 y355 w100 h30 gSave, 儲存
Gui Add, CheckBox, x165 y355 h30 vEdit_four gChangeEdit_four, 編輯
Gui, Font, s8,Microsoft JhengHei
Gui Add, Text, x90 y400 w120, © 2022 溼答答的喔

Gui, Font, s10,Microsoft JhengHei
if ( wincnt == null )
	wincnt := 1
Gui Show, w270 h420 x200 y200 , 巨集整合版-%wincnt%
Return

GuiClose:
	ExitApp
Return

;更換巨集時觸發
ChangeMacro:
	Hotkey, %action%, Action, off
	GuiControlGet, res, , MacroChoice
	GuiControlGet, res_key, , KeyChoice
	Switch res
	{
		Case "無":
			Macro := "none"
		Case "右鍵連點":
			Macro := "TOnly"
			iniWrite, %Macro%, user.ini, hotkey_setting, macro
			Hotkey, %action%, Action, on, T2
		Case "咒術-死神":
			Macro := "Warlock_third"
			iniWrite, %Macro%, user.ini, hotkey_setting, macro
			Hotkey, %action%, Action, on, T2
		Case "燐劍-風系(簡化)":
			Macro := "Blade_Dancer_wind"
			iniWrite, %Macro%, user.ini, hotkey_setting, macro
			Hotkey, %action%, Action, on, T2
		Case "天道-銀河(簡化)":
			Macro := "Astromancer_starcaster"
			iniWrite, %Macro%, user.ini, hotkey_setting, macro
			Hotkey, %action%, Action, on, T2
	}
Return

;切換巨集按鍵
ChangeKey:
	GuiControlGet, res, , KeyChoice
	Switch res
	{
		Case "左鍵":
			Hotkey, %action%, Action, off
			action := "LButton"
			Hotkey, %action%, Action, on, T2
			iniWrite, %action%, user.ini, hotkey_setting, action
		Case "右鍵":
			Hotkey, %action%, Action, off
			action := "RButton"
			Hotkey, %action%, Action, on, T2
			iniWrite, %action%, user.ini, hotkey_setting, action
		Case "側鍵1":
			Hotkey, %action%, Action, off
			action := "XButton1"
			Hotkey, %action%, Action, on, T2
			iniWrite, %action%, user.ini, hotkey_setting, action
		Case "側鍵2":
			Hotkey, %action%, Action, off
			action := "XButton2"
			Hotkey, %action%, Action, on, T2
			iniWrite, %action%, user.ini, hotkey_setting, action
	}
Return

;啟用禁用巨集
ChangeHotkeyBtn:
	GuiControlGet, res, , HotKeyBtn
	if( res == 0 ){
		HotkeyChecked := 0
		GuiControl, Disable, MacroChoice
		GuiControl, Disable, KeyChoice
		Hotkey, %action%, Action, off
		iniWrite, %HotkeyChecked%, user.ini, hotkey_setting, HotkeyChecked
	}
	Else{
		HotkeyChecked := 1
		GuiControl, Enable, MacroChoice
		GuiControl, Enable, KeyChoice
		Hotkey, %action%, Action, on, T2
		iniWrite, %HotkeyChecked%, user.ini, hotkey_setting, HotkeyChecked
	}
Return

;切換模式時觸發,按一下
ModeOne:
	mode := 1
Return

;切換模式時觸發,按住
ModeTwo:
	mode := 2
Return

;常用分頁編輯鈕
ChangeEdit_one:
	GuiControlGet, res, , Edit_one
	if ( res == 1){
		GuiControl, Enable, SpeedTimes
		GuiControl, Enable, Address
	}
	Else{
		GuiControl, Disable, SpeedTimes
		GuiControl, Disable, Address
	}
Return

;開啟加速
SpeedOn:
	set_Speed( speed )
Return

;關閉加速
SpeedOff:
	set_speed( 1 )
Return

;開關加速按鍵相同時使用
Speed:
	if ( speed_now == speed ){
		set_Speed( 1 )
		speed_now := 1
	}
	Else{
		set_Speed( speed )
		speed_now := speed
	}
Return

;禁用按鍵
Sus:
	suspend
	if( susFg == 0 ){
		ToolTip, 禁用按鍵, 900, 0
		susFg := 1
	}
	Else{
		ToolTip
		susFg := 0
	}
Return

;停止線程
Pus:
	ToolTip, 已強制停止，請重啟, 900, 0
	pause, on
Return

;重啟程式
Re:
	reload
Return

;按鍵分頁編輯鈕
ChangeEdit_two:
	GuiControlGet, res, , Edit_two
	if ( res == 1){
		GuiControl, Enable, SpeedOn
		GuiControl, Enable, SpeedOff
		GuiControl, Enable, Sus
		GuiControl, Enable, Pus
		GuiControl, Enable, Re
	}
	Else{
		GuiControl, Disable, SpeedOn
		GuiControl, Disable, SpeedOff
		GuiControl, Disable, Sus
		GuiControl, Disable, Pus
		GuiControl, Disable, Re
	}
Return

;龍脈調整用標籤
E1:
	iniRead, pX,  pos.ini, E1, x
	iniRead, pY,  pos.ini, E1, y
	iniRead, pZ,  pos.ini, E1, z
Return
E2:
	iniRead, pX,  pos.ini, E2, x
	iniRead, pY,  pos.ini, E2, y
	iniRead, pZ,  pos.ini, E2, z
Return
E3:
	iniRead, pX,  pos.ini, E3, x
	iniRead, pY,  pos.ini, E3, y
	iniRead, pZ,  pos.ini, E3, z
Return
E4:
	iniRead, pX,  pos.ini, E4, x
	iniRead, pY,  pos.ini, E4, y
	iniRead, pZ,  pos.ini, E4, z
Return
E5:
	iniRead, pX,  pos.ini, E5, x
	iniRead, pY,  pos.ini, E5, y
	iniRead, pZ,  pos.ini, E5, z
Return
E6:
	iniRead, pX,  pos.ini, E6, x
	iniRead, pY,  pos.ini, E6, y
	iniRead, pZ,  pos.ini, E6, z
Return
E7:
	iniRead, pX,  pos.ini, E7, x
	iniRead, pY,  pos.ini, E7, y
	iniRead, pZ,  pos.ini, E7, z
Return
E8:
	iniRead, pX,  pos.ini, E8, x
	iniRead, pY,  pos.ini, E8, y
	iniRead, pZ,  pos.ini, E8, z
Return
S1:
	iniRead, pX,  pos.ini, S1, x
	iniRead, pY,  pos.ini, S1, y
	iniRead, pZ,  pos.ini, S1, z
Return
S2:
	iniRead, pX,  pos.ini, S2, x
	iniRead, pY,  pos.ini, S2, y
	iniRead, pZ,  pos.ini, S2, z
Return
S3:
	iniRead, pX,  pos.ini, S3, x
	iniRead, pY,  pos.ini, S3, y
	iniRead, pZ,  pos.ini, S3, z
Return
S4:
	iniRead, pX,  pos.ini, S4, x
	iniRead, pY,  pos.ini, S4, y
	iniRead, pZ,  pos.ini, S4, z
Return
T1:
	iniRead, pX,  pos.ini, T1, x
	iniRead, pY,  pos.ini, T1, y
	iniRead, pZ,  pos.ini, T1, z
Return
T2:
	iniRead, pX,  pos.ini, T2, x
	iniRead, pY,  pos.ini, T2, y
	iniRead, pZ,  pos.ini, T2, z
Return

;飛龍脈
TP:
	set( pX, pY, pZ )
Return

;切換龍脈啟用禁用
ChangeTPBtn:
	GuiControlGet, res, , TPBtn
	if( res == 1 ){
		TPChecked := 1
		Hotkey, %TPKey%, TP, on
		GuiControl, Enable, C_E1
		GuiControl, Enable, C_E2
		GuiControl, Enable, C_E3
		GuiControl, Enable, C_E4
		GuiControl, Enable, C_E5
		GuiControl, Enable, C_E6
		GuiControl, Enable, C_E7
		GuiControl, Enable, C_E8
		GuiControl, Enable, C_S1
		GuiControl, Enable, C_S2
		GuiControl, Enable, C_S3
		GuiControl, Enable, C_S4
		GuiControl, Enable, C_T1
		GuiControl, Enable, C_T2
		iniWrite, %TPChecked%, user.ini, hotkey_setting, TPChecked
	}
	Else{
		TPChecked := 0
		Hotkey, %TPKey%, TP, off
		GuiControl, Disable, C_E1
		GuiControl, Disable, C_E2
		GuiControl, Disable, C_E3
		GuiControl, Disable, C_E4
		GuiControl, Disable, C_E5
		GuiControl, Disable, C_E6
		GuiControl, Disable, C_E7
		GuiControl, Disable, C_E8
		GuiControl, Disable, C_S1
		GuiControl, Disable, C_S2
		GuiControl, Disable, C_S3
		GuiControl, Disable, C_S4
		GuiControl, Disable, C_T1
		GuiControl, Disable, C_T2
		Loop, 14
		{
			index := A_index + 10
			GuiControl, , Button%index%, 0
		}
		iniWrite, %TPChecked%, user.ini, hotkey_setting, TPChecked
	}
Return

;龍脈分頁編輯鈕
ChangeEdit_three:
	GuiControlGet, res, , Edit_three
	if ( res == 1){
		GuiControl, Enable, TPKey
	}
	Else{
		GuiControl, Disable, TPKey
	}
Return

;修改要取色的技能
ColorOne:
	SkillFg := 1
Return
ColorTwo:
	SkillFg := 2
Return
ColorThree:
	SkillFg := 3
Return
ColorFour:
	SkillFg := 4
Return

;取色
GetPos:
	initialize()
	Switch ColorSetting
	{
		Case "Warlock_third":
			If ( SkillFg == 1 ){
				Skill_two_X := setXPos( "hc-5-x", "skill_setting_warlock_third", "Skill_two_X", "two" )
				Skill_two_Y := setYPos( "hc-5-y", "skill_setting_warlock_third", "Skill_two_Y" )
				PixelGetColor, temp, Skill_two_X, Skill_two_Y, RGB
				iniWrite, %temp%, user.ini, skill_setting_warlock_third, Skill_two_C
				MsgBox, ,提示, `            取色成功, 2
			}
			If ( SkillFg == 2 ){
				Skill_F_X := setXPos( "hc-3-x", "skill_setting_warlock_third", "Skill_F_X", "F" )
				Skill_F_Y := setYPos( "hc-3-y", "skill_setting_warlock_third", "Skill_F_Y" )
				PixelGetColor, temp, Skill_F_X, Skill_F_Y, RGB
				iniWrite, %temp%, user.ini, skill_setting_warlock_third, Skill_F_C
				MsgBox, ,提示, `            取色成功, 2
			}
			loadSetting( "skill_setting_warlock_third" )
	}
Return

;切換取色啟用禁用
ChangeGetBtn:
	GuiControlGet, res, , GetBtn
	if( res == 1 ){
		GuiControl, Enable, ColorChoice
		GuiControl, Enable, C_one
		GuiControl, Enable, C_two
		GuiControl, Enable, C_three
		GuiControl, Enable, C_four
		Hotkey, %getPosKey%, GetPos, on
	}
	Else{
		Loop, 4
		{
			index := A_index + 28
			GuiControl, , Button%index%, 0
		}
		GuiControl, hide, C_one
		GuiControl, hide, C_two
		GuiControl, hide, C_three
		GuiControl, hide, C_four
		GuiControl, Choose, ColorChoice, 0
		GuiControl, Disable, ColorChoice
		Hotkey, %getPosKey%, GetPos, off
	}
Return

;取色分頁編輯鈕
ChangeEdit_four:
	GuiControlGet, res, , Edit_four
	if ( res == 1){
		GuiControl, Enable, GetPos
	}
	Else{
		GuiControl, Disable, GetPos
	}
Return

;修改取色技能文字
SkillContent:
	GuiControlGet, res, , ColorChoice
	Switch res
	{
		Case "咒術-死神":
			GuiControl, show, C_one
			GuiControl, show, C_two
			GuiControl, text, C_one, 抓取"空間崩壞(2)"
			GuiControl, text, C_two, 抓取"黑暗波動(F)"
			ColorSetting := "Warlock_third"
		Case "召喚-幻想":
			GuiControl, show, C_one
			GuiControl, show, C_two
			GuiControl, text, C_one, 抓取"後滾翻(起身2)"
			GuiControl, text, C_two, 抓取"逃脫(起身Z)"
			ColorSetting := "summoner_third"
	}
Return

;儲存按鍵
Save:
	Hotkey, %susKey%, off
	Hotkey, %pauseKey%, off
	Hotkey, %reloadKey%, off
	GuiControlGet, susKey,, Sus
	GuiControlGet, pauseKey,, Pus
	GuiControlGet, reloadKey,, Re
	GuiControlGet, getPosKey ,, GetPos
	Hotkey, %susKey%, Sus, on
	Hotkey, %pauseKey%, Pus, on
	Hotkey, %reloadKey%, Re, on
	Hotkey, %speedOn%, ,SpeedOn, off
	Hotkey, %speedOff%, SpeedOff, off
	GuiControlGet, speedOn, , SpeedOn
	GuiControlGet, speedOff, , SpeedOff
	GuiControlGet, speed, , SpeedTimes
	If ( speedOn == speedOff )
		Hotkey, %speedOn%, Speed, on
	Else{
		Hotkey, %speedOn%, SpeedOn, on
		Hotkey, %speedOff%, SpeedOff, on
	}
	GuiControlGet, address, , Address
	Hotkey, %TPKey%, ,TP, off
	GuiControlGet, TPKey, , TPKey
	GuiControlGet, res, , TPBtn
	if( res == 1 )
		Hotkey,%TPKey%, TP, on
	Control, Uncheck, , Button5, A
	Control, Uncheck, , Button7, A
	Control, Uncheck, , Button28, A
	Control, Uncheck, , Button36, A
	iniWrite, %mode%, user.ini, hotkey_setting, mode
	iniWrite, %susKey%, user.ini, hotkey_setting, suspend
	iniWrite, %pauseKey%, user.ini, hotkey_setting, pause
	iniWrite, %reloadKey%, user.ini, hotkey_setting, reload
	iniWrite, %getPosKey%, user.ini, hotkey_setting, getPosKey
	iniWrite, %speedOn%, user.ini, hotkey_setting, SpeedOn
	iniWrite, %speedOff%, user.ini, hotkey_setting, SpeedOff
	iniWrite, %speed%, user.ini, hotkey_setting, Speed
	iniWrite, 0x%address%, pos.ini, address, address
	iniWrite, %TPChecked%, user.ini, hotkey_setting, TPChecked
	iniWrite, %HotkeyChecked%, user.ini, hotkey_setting, HotkeyChecked
	iniWrite, %TPKey%, user.ini, hotkey_setting, TPKey
	MsgBox, ,提示, `            儲存成功, 2
Return

;巨集
#MaxThreadsPerHotkey 2
Action:
	if( mode == 1 ){
		Switch Macro
		{
			Case "TOnly":
				stat := !stat
				TOnly_0()
			Case "Warlock_third":
				stat := !stat
				Warlock_third_0( Skill_two_X, Skill_two_Y, Skill_two_C, Skill_F_X, Skill_F_Y, Skill_F_C)
			Case "Blade_Dancer_wind":
				stat := !stat
				Blade_Dancer_wind_0()
			Case "Astromancer_starcaster":
				stat := !stat
				Astromancer_starcaster_0()
		}
	}
	Else{
		Switch Macro
		{
		Case "TOnly":
			TOnly( action )
		Case "Warlock_third":
			Warlock_third( Skill_two_X, Skill_two_Y, Skill_two_C, Skill_F_X, Skill_F_Y, Skill_F_C, action)
		Case "Blade_Dancer_wind":
			Blade_Dancer_wind( action )
		Case "Astromancer_starcaster":
			Astromancer_starcaster( action )
		}
	}
Return
#MaxThreadsPerHotkey 1

;輔助用計時器
onesecond:
	count += 1
Return

Warlock_third_count:
	PixelGetColor, temp, Skill_two_X, Skill_two_Y, RGB
	If ( temp == Skill_two_C ){
		send 2
	}
	MouseClick, left
	MouseClick, right
	PixelGetColor, temp, Skill_F_X, Skill_F_Y, RGB
	If ( temp == Skill_F_C )
		send f
	send v
Return
