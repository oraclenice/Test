# 瑞奥NFC钥匙SDK使用说明文档
## 1.文档说明
  - 本文档用于定义瑞奥NFC钥匙，Android SDK调用说明，提供使用操作流程和说明。

  - SDK功能概述：SDK向开发人员提供NFC钥匙读写功能。

## 2.参数对象定义
- 钥匙类型
```
public enum KeyType {
    USER_KEY("User_key", 80),/*用户钥匙*/
    SETTING_KEY("Setting_key", 18), /*设置锁ID钥匙*/
    SYSCODE_KEY("System_code_key", 17), /*设置锁系统码钥匙*/
    EVENT_KEY("Event_key", 19),/*事件钥匙*/
    LOSS_KEY("Loss_report_key", 22), /*挂失钥匙*/
    ENG_KEY("Engineering_key", -10), /*施工钥匙*/
    READLOCKID_KEY("ReadLockId_key", 32), /*锁号钥匙*/
    READLOCKNUM_KEY("ReadLockNum_key", 33), /*钥匙*/
    INSTALL_KEY("Install_key", 35),/*注册钥匙*/
    EMPTY("Empty_key", 0);/*空白钥匙*/
```

- 钥匙基本信息
```
public class KeyBasicInfo implements Serializable {
    private String version;/*钥匙版本号*/
    private String driver;/*钥匙用户*/
    private byte[] regcode;/*钥匙锁匠码*/
    private byte[] syscode;/*钥匙系统码*/
    private int keyid;/*钥匙用户编号*/
    private String type;/*钥匙类型*/
    private Calendar keytime;/*钥匙时间*/
    private String serial_number;/*钥匙序列号*/
    private Calendar old_keytime;/*钥匙旧时间*/

 ```
- 设置锁ID信息
```
public class SettingKeyInfo implements Serializable {
    private KeyBasicInfo Key;/*钥匙基本信息*/
    private int LockId;/*锁ID*/
    private int LockGourpId;/*锁组ID*/
    private List<SettingKeyInfo.Event> Evnets;/*锁组事件*/
}
```
- 设置用户钥匙信息
```
public class UserKeyInfo implements Serializable {
    private KeyBasicInfo Key;/*钥匙基本信息*/
    private Calendar MakeTime;/*钥匙制作时间*/
    private Calendar BeginTime;/*钥匙开始时间*/
    private Calendar EndTime;/*钥匙结束时间*/
    private int ValDayCount;/*钥匙验证天数*/
    private List<UserKeyInfo.TimezoneInfo> TimezoneInfo;/*钥匙时间短信息*/
    private List<UserKeyInfo.ZoneInfo> ZoneInfo;/*钥匙时间片信息*/
    private int ZoneCount;/*时间片信息*/
    private List<UserKeyInfo.CalendarInfo> CalendarInfo;/*日历表信息*/
    private UserKeyInfo.DSTInfo DST;/*钥匙夏时令信息*/
    private List<UserKeyInfo.Open> Opens;/*开门权限*/
    private List<UserKeyInfo.TempOpen> TempOpens;/*临时权限*/
    private int TempOpensCount;/*临时权限数量*/
    private List<UserKeyInfo.Event> Events;/*钥匙事件*/
```
- 设置锁号钥匙信息
```
public class ReadLockIdKeyInfo implements Serializable {
    private KeyBasicInfo Key;/*钥匙基本信息*/
    private int LockId;/*锁ID*/
    private int LockGourpId;/*锁组ID*/
```
## 3. 使用流程
### 1. 加载lib、so库文件
- 导入 rayokeynfclib.jar 文件
- 按需导入 jniLibs 中.SO库文件

### 2. 注册NFC权限
```
在AndroidManifest.xml中加入
 <uses-permission android:name="android.permission.NFC" />
 <uses-permission android:name="android.permission.VIBRATE" />
 <uses-feature
   android:name="android.hardware.nfc"
   android:required="true" />
```
### 3.初始化NFC模块
```
protected void onResume() {
          super.onResume();
          nfcAdapter.enableForegroundDispatch(this, pendingIntent, mFilters,
                  mTechLists);
          if (isFirst) {
              if (NfcAdapter.ACTION_TECH_DISCOVERED.equals(getIntent()
                      .getAction())) {
                  try {
                      processIntent(getIntent());
                  } catch (SQLException e) {
                      // TODO Auto-generated catch block
                      e.printStackTrace();
                  }
              }
              isFirst = false;
          }
      }

  private void initNFC() {
        nfcAdapter = NfcAdapter.getDefaultAdapter(this);
        if (nfcAdapter == null) {
            Toast.makeText(this, "The phone no NFC", Toast.LENGTH_SHORT).show();
            finish();
            return;
        } else if (!nfcAdapter.isEnabled()) {
            Toast.makeText(this, "Please open the NFC", Toast.LENGTH_SHORT)
                    .show();
            finish();
            return;
        }
        pendingIntent = PendingIntent.getActivity(this, 0, new Intent(this,
                getClass()).addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP), 0);
        IntentFilter ndef = new IntentFilter(NfcAdapter.ACTION_TECH_DISCOVERED);
        ndef.addCategory("*/*");
        mFilters = new IntentFilter[] { ndef };
        mTechLists = new String[][] { new String[] { IsoDep.class.getName() } };
        Intent intent = getIntent();
        resolveIntent(intent);
    }

    void resolveIntent(Intent intent) {
        String action = intent.getAction();
        if (NfcAdapter.ACTION_TECH_DISCOVERED.equals(action)) {
            Tag tagFromIntent = intent.getParcelableExtra(NfcAdapter.EXTRA_TAG);
            tagDep = IsoDep.get(tagFromIntent);
            if (tagDep != null) {
                Toast.makeText(this, "Connected the NFC Successfully",
                        Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, "Connected the NFC Failed",
                        Toast.LENGTH_SHORT).show();
            }
        }
    }
      @Override
        protected void onNewIntent(Intent intent) {
            // TODO Auto-generated method stub
            super.onNewIntent(intent);
            if (NfcAdapter.ACTION_TECH_DISCOVERED.equals(intent.getAction())) {
                resolveIntent(intent);
                try {
                    processIntent(intent);
                } catch (SQLException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }

            }
        }
      /*获取tab标签中的内容*/
     private String processIntent(Intent intent) throws SQLException {
            String resultStr = "";
            Tag tagFromIntent = intent.getParcelableExtra(NfcAdapter.EXTRA_TAG);
            tagDep = IsoDep.get(tagFromIntent);
             /*验证钥匙*/
            KeyBasicInfo basicKey = new KeyBasicInfo(KeyCode.DEFREGCODE.getCode(),
            				KeyCode.DEFSYSCODE.getCode());
            		CheckKeyAction key = new CheckKeyAction(tagDep, basicKey);
                          if (KeyCMD.SUCCESS.getCMD() == key.checkKey()) {
                               ··· ···
                          }
```
### 4.NFC钥匙操作
- 设置设置钥匙
```
  private boolean setSettingKey(IsoDep dep, CheckKeyAction basic) {
        basicKey.setKeyid(userid);
        SettingKeyInfo info = new SettingKeyInfo(basicKey, lockid, groudid);
        SettingKeyAction settingKey = new SettingKeyAction(dep, info);
        if (KeyCMD.SUCCESS.getCMD() == settingKey.SetKey()) {
            ··· ···
            return true;
        } else
            return false;
    }
```
- 设置用户钥匙
```
 private boolean setUserKey(IsoDep dep, CheckKeyAction basic) {
        basicKey.setKeyid(userid);
        UserKeyInfo info = new UserKeyInfo(basicKey);
        info.setMakeTime(Calendar.getInstance());
        info.setBeginTime(BeginTime);
        info.setEndTime(EndTime);
        info.setValDayCount(valday);

        // TempOpens MAX：100
        List<UserKeyInfo.TempOpen> tempList = new ArrayList<UserKeyInfo.TempOpen>();
        UserKeyInfo.TempOpen topen = null;
        for (int temp = 0; temp < list.size(); temp++) {
             topen = new UserKeyInfo.TempOpen(temp + 1,dete,begin,end,list.get(temp));
            tempList.add(topen);
        }
        info.setTempOpens(tempList);
        info.setTempOpensCount(tempList.size());
        //
        UserKeyAction userKey = new UserKeyAction(dep, info);
        userKey.IsBeginTime = true;
        userKey.IsCalendar = true;
        userKey.IsDST = true;
        userKey.IsEndTime = true;
        userKey.IsMakeTime = true;
        userKey.IsOpens = true;
        userKey.IsTempOpens = true;
        userKey.IsTimeZone = true;
        userKey.IsValDayCount = true;
        if (KeyCMD.SUCCESS.getCMD() == userKey.SetKey()) {
            ··· ···
            return true;
        }
        return false;
    }
```
- 设置事件钥匙
```
  private boolean setEventKey(IsoDep dep, CheckKeyAction basic) {
        basicKey.setKeyid(userid);
        EventKeyInfo info = new EventKeyInfo(basicKey);
        EventKeyAction eventKey = new EventKeyAction(dep, info);
        if (KeyCMD.SUCCESS.getCMD() == eventKey.SetKey()) {
            ··· ···
            return true;
        }
        return false;
    }
```
- 设置锁号钥匙
```
    private boolean setReadLockIdKey(IsoDep dep, CheckKeyAction basic) {
        basicKey.setKeyid(userid);
        ReadLockIdKeyInfo info = new ReadLockIdKeyInfo(basicKey);
        ReadLockIdKeyAction codeKey = new ReadLockIdKeyAction(dep, info);
        if (KeyCMD.SUCCESS.getCMD() == codeKey.SetKey()) {
            ··· ···
            return true;
        }
        return false;
    }
```
- 设置锁出厂编号
```
private boolean setReadLockFactoryIdKey(IsoDep dep, CheckKeyAction basic) {
		basicKey.setKeyid(1);
		ReadLockNumKeyInfo info = new ReadLockNumKeyInfo(basicKey);

		ReadLockNumKeyAction codeKey = new ReadLockNumKeyAction(dep, info);
		if (KeyCMD.SUCCESS.getCMD() == codeKey.SetKey()) {
			··· ···
			return true;
		}
		return false;
	}
```
- 设置施工钥匙
```
private boolean setEngineeringkey(IsoDep dep, CheckKeyAction basic) {
		basicKey.setKeyid(1);
		EngineeringKeyInfo info = new EngineeringKeyInfo(basicKey);
		Calendar nowdate = Calendar.getInstance();
		info.setMakeTime(Calendar.getInstance());
		info.setBeginTime(Calendar.getInstance());
		nowdate.add(Calendar.MONTH, 1);
		info.setEndTime(nowdate);
		info.setValDayCount(30);
		EngineeringKeyAction engKey = new EngineeringKeyAction(dep, info);
		if (KeyCMD.SUCCESS.getCMD() == engKey.SetKey()) {
			··· ···
			return true;
		} else
			return false;
	}
```
- 设置挂失钥匙
```
	private boolean setLossKey(IsoDep dep, CheckKeyAction basic) {
		basicKey.setKeyid(1);
		LossKeyInfo info = new LossKeyInfo(basicKey);
		int[] usrList = new int[10];
		for (int i = 0; i < 10; i++) {
			usrList[i] = i + 10;
		}
		info.setUserId(usrList);
		LossKeyAction lossKey = new LossKeyAction(dep, info);
		if (KeyCMD.SUCCESS.getCMD() == lossKey.SetKey()) {
			··· ···
			return true;
		}
		return false;
	}
```
- 设置注册钥匙
```
private boolean setInstallKey(IsoDep dep, CheckKeyAction basic) {
		basicKey.setKeyid(1);
		InstallKeyInfo info = new InstallKeyInfo(basicKey);
		Calendar nowdate = Calendar.getInstance();
		info.setMakeTime(Calendar.getInstance());
		info.setBeginTime(Calendar.getInstance());
		nowdate.add(Calendar.MONTH, 1);
		info.setEndTime(nowdate);
		InstallKeyAction codeKey = new InstallKeyAction(dep, info);
		if (KeyCMD.SUCCESS.getCMD() == codeKey.SetKey()) {
			··· ···
			return true;
		}
		return false;
	}
```
- 读用户钥匙
```
if (key.getType().equals(KeyType.USER_KEY.getName())) {
            UserKeyInfo info = new UserKeyInfo(basicKey);
            UserKeyAction userKey = new UserKeyAction(dep, info);
            if (KeyCMD.SUCCESS.getCMD() == userKey.GetKey()) {
```
- 读设置钥匙
```
 if (key.getType().equals(KeyType.SETTING_KEY.getName())) {
            SettingKeyInfo info = new SettingKeyInfo(basicKey);
            SettingKeyAction settingKey = new SettingKeyAction(dep, info);
            if (KeyCMD.SUCCESS.getCMD() == settingKey.GetKey()) {
```
- 读事件钥匙
```
if (key.getType().equals(KeyType.EVENT_KEY.getName())) {
			EventKeyInfo info = new EventKeyInfo(basicKey);
			EventKeyAction eventKey = new EventKeyAction(dep, info);
			if (KeyCMD.SUCCESS.getCMD() == eventKey.GetKey()) {
```
- 读施工钥匙
```
if (key.getType().equals(KeyType.ENG_KEY.getName())) {
			EngineeringKeyInfo info = new EngineeringKeyInfo(basicKey);
			EngineeringKeyAction engKey = new EngineeringKeyAction(dep, info);
			if (KeyCMD.SUCCESS.getCMD() == engKey.GetKey()) {
```
- 读挂失钥匙
```
if (key.getType().equals(KeyType.LOSS_KEY.getName())) {
			LossKeyInfo info = new LossKeyInfo(basicKey);
			LossKeyAction lossKey = new LossKeyAction(dep, info);
			if (KeyCMD.SUCCESS.getCMD() == lossKey.GetKey()) {
```
- 读锁号钥匙
```
if (key.getType().equals(KeyType.READLOCKNUM_KEY.getName())) {
			ReadLockNumKeyInfo info = new ReadLockNumKeyInfo(basicKey);
			ReadLockNumKeyAction codeKey = new ReadLockNumKeyAction(dep, info);
			if (KeyCMD.SUCCESS.getCMD() == codeKey.GetKey()) {
```
- 读锁出厂编号
```
if (key.getType().equals(KeyType.READLOCKNUM_KEY.getName())) {
			ReadLockNumKeyInfo info = new ReadLockNumKeyInfo(basicKey);
			ReadLockNumKeyAction codeKey = new ReadLockNumKeyAction(dep, info);
			if (KeyCMD.SUCCESS.getCMD() == codeKey.GetKey()) {
```
- 读注册钥匙
```
if (key.getType().equals(KeyType.INSTALL_KEY.getName())) {
			InstallKeyInfo info = new InstallKeyInfo(basicKey);
			InstallKeyAction codeKey = new InstallKeyAction(dep, info);
			if (KeyCMD.SUCCESS.getCMD() == codeKey.GetKey()) {
```
## 5. 事件列表
```
public enum KeyEventType {
    EVT_OPEN_SUCCESS("Open Successfully", 1),//开门成功
    EVT_SETT_SUCCESS("Setting Lock ID Successfully", 2),//设置锁成功
    EVT_SETT_FAILURE("Setting Lock ID Failed", 3),//设置锁失败
    EVT_OUT_VALIDITY("Out of the validity date", 4),//钥匙超出有限期
    EVT_NO_RIGHTOPEN("No authority to open", 5),//钥匙没有权限开门
    EVT_OUT_TIMEZONE("Out of the timezone", 6),//钥匙超出时间片范围
    EVT_SYSCODEERROR("System code error", 7),//钥匙系统码错误
    EVT_BLACKK_LISTT("The user is in Black List", 8),//钥匙在黑名单中
```
## 6. 钥匙指令
```
public enum KeyCMD {
    SUCCESS("success", 2),//成功
    FAILED("failed", 1),//失败
    CHECK("check", 0),//验证
    VERSION("version", 17),//版本号
    DRIVER("driver", 18),//用户
    ID("id", 19),//编号
    TYPE("type", 20),//类型
    REGCODE("regcode", 21),//锁匠码
    SYSCODE("syscode", 22),//系统码
    GROUPID("groupid", 23),//组编号
    MAKETIME("maketime", 24),//制作时间
    ENDTIME("endtime", 25),//截止时间
    BEGINTIME("begintime", 26),//开始时间
    TIMEZONE_COUNT("timezonecount", 27),//时间片数量
    TIMEINFO_COUNT("timeinfocount", 28),//时间短信息数量
    TIMEINFO("timeinfo", 29),//时间短信息
    CLEAN_OPEN("cleanopen", 30),//清空开锁权限
    ADD_OPEN("add-open", 31),//添加权限
    DEL_OPEN("del-open", 47),//删除权限
    EVENT_COUNT("eventcount", 32),//事件数量
    EVENT_ADD("eventadd", 33),//添加事件
    EVENT("event", 48),//事件
    TEMPCOUNT("tempcount", 34),//临时权限数量
    TEMP("temp", 35),//临时权限
    CALENDAR("calendar", 36),//日历表信息
    DST("dst", 37),//夏时令信息
    VALDAY_COUNT("valdaycount", 38),//验证天数
    LOCK_SYSCODE("locksyscode", 39),//锁系统编码
    LOCK_ID("lockid", 40),//锁编号
    LOCK_GROUP("lockgroup", 41),//锁组号
    LOCK_EVENT_COUNT("lockeventcount", 42),//锁事件数量
    LOCK_EVENT_INFO("lockeventinfo", 43),//锁时间信息
    LOCK_EVENT("lockevent", 44),//锁事件
    LOSS_KEY_COUNT("losskeycount", 45),//挂失钥匙数量
    LOSS_KEY("losskey", 46),//挂失钥匙
    RESTART_KEY("restartkey", -1),//重启钥匙
    RESET_KEY("resetkey", -2),//重置钥匙
    USER_BLACK_FLAG("userblackflag", 49),//黑名单用户
    SERIAL_NUMBER("serialnumber", 51),//序列号
```
