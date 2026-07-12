##1. windows预口体验成员内口版本遇到问题需要重启,无法启动
解决：https://answers.microsoft.com/zh-hans/windows/forum/windows_10-other_settings/windows%E9%A2%84%E5%8F%A3%E4%BD%93%E9%AA%8C/8221aa34-a529-4fe7-8ab7-02b62c06bd42
步骤：
1. 按键盘Win+R打开"运行",输入cmd然后按Ctrl+Shift+回车,以管理员身份打开命令提示符.或者按Win+S在搜索框中输入cmd,右键点击搜索结果中的"命令提示符">以管理员身份运行

2. 在打开的命令提示符窗口中粘贴运行下面的命令:

DISM.exe /Online /Cleanup-image /Restorehealth

sfc /scannow

3. 无法解决，重装系统，最简单
