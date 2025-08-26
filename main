#include <GUIConstantsEx.au3>
#include <MsgBoxConstants.au3>
#include <WindowsConstants.au3>
#include <EditConstants.au3>
#include <ButtonConstants.au3>
#include <ComboConstants.au3>
#include <StaticConstants.au3>
#include <FileConstants.au3>
#include <Misc.au3>

Global $inputNumber, $chooseFile, $msgInput, $intervalInput, $comboMode, $startButton, $countdownLabel, $chkRandom
Global $g_sFilePath = ""
Global $fileLabel

; GUI
GUICreate("Otomatis WA Sender", 420, 380)

GUICtrlCreateLabel("Mode:", 10, 10, 60, 20)
$comboMode = GUICtrlCreateCombo("", 80, 10, 150, 20)
GUICtrlSetData($comboMode, "Single Number|Bulk Number", "Single Number")

GUICtrlCreateLabel("Nomor WA:", 10, 50, 70, 20)
$inputNumber = GUICtrlCreateInput("", 80, 50, 200, 20)

$chooseFile = GUICtrlCreateButton("Choose File", 300, 50, 100, 25)
$fileLabel = GUICtrlCreateLabel("File: (belum dipilih)", 80, 75, 320, 20)

GUICtrlCreateLabel("Pesan: (Gunakan {name} untuk nama)", 10, 100, 300, 20)
$msgInput = GUICtrlCreateEdit("", 10, 120, 390, 100, BitOR($ES_AUTOVSCROLL, $ES_WANTRETURN, $WS_VSCROLL))

GUICtrlCreateLabel("Interval (detik):", 10, 230, 100, 20)
$intervalInput = GUICtrlCreateInput("5", 120, 230, 50, 20)

$chkRandom = GUICtrlCreateCheckbox("Random Interval", 200, 230, 200, 20)

$countdownLabel = GUICtrlCreateLabel("", 10, 260, 400, 20)

$startButton = GUICtrlCreateButton("Start", 160, 310, 100, 30)

GUISetState(@SW_SHOW)
_UpdateMode()

; MAIN LOOP
While 1
    Switch GUIGetMsg()
        Case $GUI_EVENT_CLOSE
            Exit
        Case $comboMode
            _UpdateMode()
        Case $chooseFile
            Local $sFile = FileOpenDialog("Pilih file berisi nomor WA", @ScriptDir, "Text Files (*.txt)", $FD_FILEMUSTEXIST)
            If @error = 0 Then
                $g_sFilePath = $sFile
                GUICtrlSetData($fileLabel, "File: " & StringTrimLeft($sFile, StringInStr($sFile, "\", 0, -1)))
            EndIf
        Case $chkRandom
            If GUICtrlRead($chkRandom) = $GUI_CHECKED Then
                GUICtrlSetState($intervalInput, $GUI_DISABLE)
            Else
                GUICtrlSetState($intervalInput, $GUI_ENABLE)
            EndIf
        Case $startButton
            _StartSend()
    EndSwitch
WEnd

; FUNCTION 
Func _UpdateMode()
    Local $mode = GUICtrlRead($comboMode)
    If $mode = "Single Number" Then
        GUICtrlSetState($chooseFile, $GUI_DISABLE)
        GUICtrlSetState($fileLabel, $GUI_DISABLE)
        GUICtrlSetState($inputNumber, $GUI_ENABLE)
    Else
        GUICtrlSetState($chooseFile, $GUI_ENABLE)
        GUICtrlSetState($fileLabel, $GUI_ENABLE)
        GUICtrlSetState($inputNumber, $GUI_DISABLE)
    EndIf
EndFunc

Func _StartSend()
    Local $interval
    If GUICtrlRead($chkRandom) = $GUI_CHECKED Then
        $interval = Random(12, 47, 1)
    Else
        $interval = Int(GUICtrlRead($intervalInput))
        If $interval <= 0 Then $interval = 5
    EndIf

    For $i = 3 To 1 Step -1 ; Countdown 
        GUICtrlSetData($countdownLabel, "Mulai dalam: " & $i & " detik")
        Sleep(1000)
    Next
    GUICtrlSetData($countdownLabel, "Memulai proses...")

    Local $mode = GUICtrlRead($comboMode)
    Local $message = GUICtrlRead($msgInput)

    If $mode = "Single Number" Then
        Local $number = GUICtrlRead($inputNumber)
        If $number = "" Then
            MsgBox($MB_ICONERROR, "Error", "Isi nomor dulu bro.")
            Return
        EndIf
        _OpenWA($number, $message)
        GUICtrlSetData($countdownLabel, "Selesai!")
    Else
        If $g_sFilePath = "" Then
            MsgBox($MB_ICONERROR, "Error", "Pilih file nomor WA dulu, bro.")
            Return
        EndIf
        Local $fh = FileOpen($g_sFilePath, $FO_READ)
        If $fh = -1 Then
            MsgBox($MB_ICONERROR, "Error", "Gagal buka file: " & $g_sFilePath)
            Return
        EndIf
        While 1
            Local $line = FileReadLine($fh)
            If @error = -1 Then ExitLoop
            Local $aData = StringSplit(StringStripWS($line, 8), ",")
            If $aData[0] = 2 Then
                Local $sName = $aData[1]
                Local $sNumber = $aData[2]
                Local $sPersonalizedMessage = StringReplace($message, "{name}", $sName)
                GUICtrlSetData($countdownLabel, "Mengirim ke " & $sName & "...")
                _OpenWA($sNumber, $sPersonalizedMessage)

                If GUICtrlRead($chkRandom) = $GUI_CHECKED Then
                    $interval = Random(12, 47, 1)
                Else
                    $interval = Int(GUICtrlRead($intervalInput))
                    If $interval <= 0 Then $interval = 5
                EndIf
                For $i = $interval To 1 Step -1
                    GUICtrlSetData($countdownLabel, "Nomor selanjutnya dalam: " & $i & " detik")
                    Sleep(1000)
                Next
            EndIf
        WEnd
        FileClose($fh)
        GUICtrlSetData($countdownLabel, "Selesai! Semua nomor dari file telah diproses.")
    EndIf
EndFunc

Func _OpenWA($number, $message)
    Local $sEncodedMessage = _URIEncode($message)
    Local $url = "https://wa.me/" & StringStripWS($number, 8) & "?text=" & $sEncodedMessage
    ShellExecute($url)

    Local $hWnd = WinWaitActive("WhatsApp", "", 30)

    If $hWnd Then
        Sleep(3000)

        Send("{ENTER}") ; press Enter after countdown
        Sleep(500) 
    Else

        GUICtrlSetData($countdownLabel, "Gagal menemukan window WhatsApp/Browser.")
        Sleep(2000)
    EndIf
EndFunc

Func _URIEncode($sString)
    Local $sResult = ""
    For $i = 1 To StringLen($sString)
        Local $sChar = StringMid($sString, $i, 1)
        If StringRegExp($sChar, "[a-zA-Z0-9-_.!~*'()]") Then
            $sResult &= $sChar
        ElseIf $sChar = " " Then
            $sResult &= "%20" 
        Else
            $sResult &= "%" & Hex(Asc($sChar), 2)
        EndIf
    Next
    Return $sResult
EndFunc
