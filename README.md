# 瑞奥离线门控器SDK使用说明文档
## 1.文档说明
  - 本文档用于定义瑞奥离线门控器，Android SDK调用说明，提供使用操作流程和说明。

  - SDK功能概述：SDK向开发人员提供离线门控器读写功能。

## 2.参数对象定义

- 离线门控器基本信息
```
public class DmuInfo {
	private String version;//版本号
	private String driver;//用户
	private byte[] regcode = KeyCode.DEFREGCODE.getCode();//锁匠码
	private byte[] syscode = KeyCode.DEFSYSCODE.getCode();//系统码
	private int dmuid;//离线门控器编号
	private int groupid;//离线门控器组号
	private int mark;//标记
	private Calendar dmutime;//时间
	private Calendar old_dmutime;//旧时间
	private List<Event> Events = new ArrayList<Event>();//事件
	private List<byte[]> DmuUsers = new ArrayList<byte[]>();//用户列表
	private int DmuUserStart;//用户起始列表


 ```

## 3. 使用流程
### 1. 加载lib、so库文件
- 导入 lib库 文件
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
             /*验证*/
                ··· ···
```
### 4.离线门控器操作
- 验证离线门控器
```
 DmuInfo dmuInfo = new DmuInfo(byte[] Regcode, byte[] Syscode);
 dmuInfo.setDmutime(Calendar.getInstance());
 CheckDmuAction dmu = new CheckDmuAction(tagDep, dmuInfo);
```
- 设置离线门控器ID信息
```
private String MakeDmu(DmuInfo dmuInfo) {
            /*设置离线标记*/
            dmuInfo.setMark(mark);
            SetMarkAction setMarkAction =new SetMarkAction(tagDep, dmuInfo);
            setMarkAction.SetDmu();
            /*设置id*/
            dmuInfo.setDmuid(lockid);
            /*设置组id*/
            dmuInfo.setGroupid(groudid);
            SetDmuAction setid = new SetDmuAction(tagDep, dmuInfo);
            if (DmuCMD.SUCCESS.getCMD() == setid.SetDmu()) {
```
- 更新用户列表
```
dmuInfo.setDmuUsers(userArray);
					dmuInfo.setDmuUserStart(no);
					UpdateDmuUserListAction userList = new UpdateDmuUserListAction(
							tagDep, dmuInfo);
					userList.SetDmu();
```
- 读离线门控器信息
```
	ReadDmuInfoAction readDmu = new ReadDmuInfoAction(tagDep, dmuInfo);
			readDmu.readDmuInfo();
```
- 读离线门控器事件
```
	ReadEventAction event = new ReadEventAction(tagDep, dmuInfo);
			event.ReadEventInfo();
```

## 5. 事件列表
```
public enum KeyEventType {
    EVT_OPEN_SUCCESS("Open Successfully", 1),//开门成功
    EVT_SETT_SUCCESS("Setting Lock ID Successfully", 2),//设置离线门控器成功
    EVT_SETT_FAILURE("Setting Lock ID Failed", 3),//设置离线门控器失败
    EVT_OUT_VALIDITY("Out of the validity date", 4),//钥匙超出有限期
    EVT_NO_RIGHTOPEN("No authority to open", 5),//钥匙没有权限开门
    EVT_OUT_TIMEZONE("Out of the timezone", 6),//钥匙超出时间片范围
    EVT_SYSCODEERROR("System code error", 7),//钥匙系统码错误
    EVT_BLACKK_LISTT("The user is in Black List", 8),//钥匙在黑名单中
    EVT_OPEN_SUCCESS_READ2("Open Successfully Reader2", 49),
    EVT_SETT_SUCCESS_READ2("Setting Lock ID Successfully Reader2", 50),
    EVT_SETT_FAILURE_READ2("Setting Lock ID Failed Reader2", 51),
    EVT_OUT_VALIDITY_READ2("Out of the validity date Reader2", 52),
    EVT_NO_RIGHTOPEN_READ2("No authority to open Reader2", 53),
    EVT_OUT_TIMEZONE_READ2("Out of the timezone Reader2", 54),
    EVT_SYSCODEERROR_READ2("System code error Reader2", 55),
    EVT_BLACKK_LISTT_READ2("The user is in Black List Reader2", 56),
    EVT_OPEN_SUCCESS_CARD1("Open Successfully by card", 97),
    EVT_SETT_SUCCESS_CARD1("Setting Lock ID Successfully by card", 98),
    EVT_SETT_FAILURE_CARD1("Setting Lock ID Failed by card", 99),
    EVT_OUT_VALIDITY_CARD1("Out of the validity date by card", 100),
    EVT_NO_RIGHTOPEN_CARD1("No authority to open by card", 101),
    EVT_OUT_TIMEZONE_CARD1("Out of the timezone by card", 102),
    EVT_SYSCODEERROR_CARD1("System code error by card", 103),
    EVT_BLACKK_LISTT_CARD1("The user is in Black List by card", 104),
    EVT_OPEN_SUCCESS_CARD2("Open Successfully Reader2 by card", -111),
    EVT_SETT_SUCCESS_CARD2("Setting Lock ID Successfully Reader2 by card", -110),
    EVT_SETT_FAILURE_CARD2("Setting Lock ID Failed Reader2 by card", -109),
    EVT_OUT_VALIDITY_CARD2("Out of the validity date Reader2 by card", -108),
    EVT_NO_RIGHTOPEN_CARD2("No authority to open Reader2 by card", -107),
    EVT_OUT_TIMEZONE_CARD2("Out of the timezone Reader2 by card", -106),
    EVT_SYSCODEERROR_CARD2("System code error Reader2 by card", -105),
    EVT_BLACKK_LISTT_CARD2("The user is in Black List Reader2 by card", -104);
```
## 6. 离线门控器指令
```
public enum DmuCMD {
    SUCCESS("success", (byte)0x02),//成功
	FAILED("failed",(byte)0x01),//失败
	CHECK("check", (byte) 0x0),//验证
	VERSION("version", (byte) 0x11),//版本号

	ID("id",(byte) 0x13),//编号
	REGCODE("regcode", (byte) 0x15),//锁匠码
	SYSCODE("syscode", (byte) 0x16),//系统码
	DMUTIME("maketime", (byte) 0x17),//时间

	TAGER_IP("tagerip",(byte) 0x18),//
	KEY_BLACK("keyblack",(byte) 0x19),//黑名单钥匙
	CARD_BLACK("cardblack",(byte) 0x20),//黑名单卡片
	OPEN_MODE("openmode",(byte) 0x21),//
	OPEN_TIMEZONE("opentimezone",(byte) 0x22),//临时权限
	EVENT_INFO("eventinfo",(byte) 0x23),//事件信息
	EVENT("event",(byte) 0x24),//事件
	CLEAN_EVENT("cleanevent",(byte) 0x25),清空事件
	VAL_USERLIST("valuserlist",(byte) 0x26),//用户列表
	CLEAN_VAL_USERLIST("cleanvaluserlist",(byte) 0x27),//清空用户列表
	VAL_COUNT("valuserlistcount",(byte)0x28),//验证天数
	VERSION_NO("versionno",(byte)0x29),//版本号
	PRODUCT_NO("productno",(byte)0x30),//生产编号
	MARK("mark",(byte)0x31),//离线标记

	RESTART_DMU("restartdmu",(byte)0xff),//重启离线门控器
	RESET_DMU("resetdmu",(byte)0xfe);//重置离线门控器

```
