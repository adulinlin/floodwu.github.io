# 《Android编程权威指南》读书笔记


## Android编译过程
 ![image](https://github.com/woojean/woojean.github.io/blob/master/images/android_1.png)

## Android的MVC设计模式
`Model层`：存储应用的数据和业务逻辑，通常被设计来映射与应用相关的一些事物，存在的唯一目的是存储和管理应用数据。
`View层`：知道如何在屏幕上绘制自己以及如何响应用户的输入，如触摸等。通常由布局文件中定义的各类组件构成。
`Controller层`：用来响应由视图对象触发的各类事件，以及管理模型对象与视图层间的数据流动。在Android中，控制器通常是Activity、Fragment或Service的一个子类。

## Activity生命周期 
设备旋转时，当前看到的Activity实例会被系统销毁，然后重新创建。因为设备旋转会改变设备配置，所谓设备配置即用来描述设备当前状态的一系列特征，包括：屏幕的方向、屏幕的密度、屏幕的尺寸、键盘类型、底座模式、语言等等。通常为匹配不同的设备配置，应用会提供不同的备选资源，在运行时，当设备配置发生变更时，会销毁当前Activity并重建。
为了保存设备旋转以前的数据，需要覆盖`onSaveInstanceState(Bundle outState)`方法，将一些数据保存在Bundle中，然后在onCreate()方法中取回这些数据。注意：在Bundle中存储和恢复的数据类型只能是基本数据类型以及可以实现Serializable接口的对象。 
onSaveInstanceState方法通常在onPause()、onStop()、onDestory()方法之前由系统调用，并不仅仅用来处理设备配置变更的问题，当用户离开当前activity管理的用户界面，或Activity需要回收内存时，activity也会被销毁。在描述Activity的生命周期时，需要将其考虑进来：

![image](https://github.com/woojean/woojean.github.io/blob/master/images/android_2.png)

`Activity记录`：当调用onSaveInstanceState方法时，用户数据会被保存在Bundle对象中，然后操作系统将Bundle对象放入Activity记录中，在需要恢复Activity时，操作系统可以使用暂存的Activity记录重新激活Activity。即使用户离开当前应用（此时彻底停止了当前应用的进程），暂存的Activity记录依然被系统保留。除非用户通过按后退键退出应用，或者系统重启，或者长时间不适用Activity，此时系统彻底销毁当前Activity，暂存的Activity记录通常也会被清除。

## Activity通信
activity调用startActivity(...)方法时，调用请求实际发给了ActivityManager，ActivityManager负责创建Activity实例，并调用其onCreate()方法。

Intent对象是Android Component（Activity、Service、Broadcast Receiver、Content Provider）之间用来通信的一种媒介工具。如果通过指定Context与Class对象的方式来创建Intent，则创建的是显式Intent，否则就是隐式Intent（指定动作和标志）。显式Intent通常用于同一个应用内的通信，隐式Intent通常用于不同应用间的通信。

启动Activity-不带参数：
```java
Intent i = new Intent(XxxActivity.this,YyyActivity.class);
startActivity(i);
```
启动Activity-带参数：
```java
Intent i = new Intent(XxxActivity.this,YyyActivity.class);
boolean param = true;
i.putExtra(“PARAM_NAME”,param);
startActivity(i);
```
被启动Activity获取参数：
```java
@Override
protected void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
mParam = getIntent().getBooleanExtra(“PARAM_NAME”);
```
启动Activity后，获取被启动Activity的返回值：
启动：
```java
Intent i = new Intent(XxxActivity.this,YyyActivity.class);
boolean param = true;
i.putExtra(“PARAM_NAME”,param);
startActivityForResult(i,REQUEST_CODE);
```
被启动Activity返回：
```java
Intent i= new Intent();
i.putExtra(“RETURN_VALUE”,mReturn);
setResult(RESULT_OK,i);  // 另一个可取标记为Activity.RESULT_CANCELED
```
接收返回值：
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
if(data == null){
return;
}
mParam = data.getBooleanExtra(“PARAM_NAME”,false);
}
```
注意这里onActivityResult(...)方法的签名中既需要启动时的requestCode，也需要返回时的resultCode。

ActivityManager维护着一个非特定应用独享的回退栈，所有应用的activity都共享该回退栈。即ActivityManager被设计成操作系统级的activity管理器来负责启动activity，不局限于单个应用，回退栈作为一个整体共享给操作系统及设备使用。当用户点击一个应用时，实际是启动了该应用的launcher activity，并将它加入到activity栈中。

## SDK版本
SDK最低版本：操作系统会拒绝将应用安装在系统版本低于该标准的设备上。
SDK目标版本：告知系统该应用是给哪个API级别去运行的，高于目标版本的系统功能将被忽略。
SDK编译版本：该设置不会出现在Manifest文件中（不同于最低版本与目标版本），不会通知给操作系统，而是用于编译时指定具体要使用的系统版本。

检查设备的Android系统的编译版本：
if( Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB ){
...
}

禁止Lint提示兼容性问题：
```java
@TargetApi(11)
@Override
protected void onCreate(Bundle savedInstanceState){
...
if( Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB ){
...
}
}
```
## Fragment的使用
Fragment在API 11以后才被引入到标准库。之前的版本要想使用Fragment，必须使用Android支持库中的android.support.v4.app.Fragment并配合android.support.v4.app.FragmentActivity类来使用。

Fragment不能单独呈现UI，他必须托管于Activity才能使用，有两种托管Fragment的方式：
1.添加fragment到activity布局中；（布局文件方式，在Activity的生命周期中无法切换fragment视图）
2.在activity代码中添加fragment；（代码方式，可以在运行时控制fragment）

添加Fragment到Activity的步骤：
1.定义Fragment的视图文件
2.新建继承于Fragment的自定义类，重写其onCreateView方法，渲染布局文件，设置布局中各个控件的回调函数，最后返回该视图：
```java
public class CrimeFragment extends Fragment{
@Override
public View onCreateView(LayoutInflater inflater,ViewGroup parent,Bundle savedInstanceState){
View v = inflater.inflate(R.layout.fragment_crime,parent,false);
mTitleField = (EditText)v.findViewById(R.id.xxx);
mTitleField.addTextChangedListener(
new TextWatcher(){
public void onTextChanged(CharSequence c, int start, int before, int count){
...
};
}
);
return v;
}
}
```
3.在Activity中通过FragmentManager将Fragment添加到Activity的布局中：
```java
public class CrimeActivity extends FragmentActivity{
@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_xxx);

FragmentManager fm = getSupportFragmentManager();
Fragment fragment = fm.findFragmentById(R.id.fragmentContainer);
if(fragment == null){ // 判断是否已存在该Fragment
fragment = new CrimentFragment();
fm.beginTransaction().add(R.id.fragmentContainer,fragment).commit();
}
}
}
```
## Fragment的生命周期
![image](https://github.com/woojean/woojean.github.io/blob/master/images/android_3.png)

托管Activity的onCreate()方法执行之后，Fragment的onActivityCreated(...)方法也会被调用。在Activity处于停止、暂停或运行状态下时，FragmentManager立即驱使fragment跟上activity的步伐（维护两者的状态一致），直到与activity的最新状态保持同步。比如向处于运行状态的activity中添加fragment时，以下fragment生命周期的方法会被依次调用：onAttach()、onCreate()、onCreateView()、onActivityCreated()、onStart()、onResume()。

## android:layout_weight属性
layout_weight属性用于LinearLayout进行子组件的布局安排。在决定子组件视图的宽度（横向布局时）时，LinearLayout使用的是layout_width与layout_weight参数的混合值，具体分为以下两步：
1.LinearLayout查看layout_weight属性值（竖直方向则是layout_height属性），比如两个空间的layout_width都设置为wrap_content，则依他们的实际内容的宽度绘制控件。
2.LinearLayout依据layout_weight属性值进行额外的空间分配，若两者layout_weight属性相同，则均分剩余空间，如一个空间layout_weight为2，另一个为1，则为2的控件获得2/3的剩余空间，为1的控件获得1/3的剩余空间。
如果想让控件占据相同的宽度，可以将layout_weight设为0dp，将layout_weight设为一样。

使用ListFragment显示列表
自定义一个继承于android.v4.app.ListFragment的类，该类默认生成一个ListView布局，因此无需覆盖onCreateView()方法，或为其生成布局。

创建一个托管该ListFragment的Activity类，方法与添加一般的Fragment一样。

在自定义ListFragment的onCreate()方法中实现一个ArrayAdapter<T>，并使用setListAdapter()方法进行设置。实现ArrayAdapter有两种方式，一是通过指定Item的视图文件来定义每一项的视图，二是自定一个继承自ArrayAdapter的类，重写其getView()方法，该方法基于实际的数据集来映射出具体的视图。

重写ListFragment的onListItemClick()方法，设置点击列表项时的响应事件。该方法会传入一个position参数用以判断所点击的具体项。

之后，数据有更新时，则修改底层getview()方法所使用的数据集，然后调用ListAdapter的notifyDataSetChanged()方法,通常在Fragment的onResume()方法中调用。
## Fragment Argument
每个Fragment实例都可附带一个Bundle对象，该bundle对象包含key-value对，一个key-value对即是一个argument。通过调用Fragment.setArguments(Bundle)方法进行设置，该任务必须在fragment创建后、添加给activity之前完成。通常的做法是实现一个newInstance()方法来封装以上行为：
```java
public class CrimeFragment extends Fragment{

@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
// 获取argument
UUID crimeId = (UUID)getArguments.getSerializable(EXTRA_CRIME_ID); 
}

public static CrimeFragment newInstance(UUID crimeId){
Bundle args = new Bundle();
args.putSerializable(EXTRA_CRIME_ID,crimeId);
CrimeFragment fragment = new CrimeFragment();
fragment.setArgument(args);
return fragment;
}
...
}
```
在Activity中调用newInstance()方法
```java
public class CrimeActivity extends SingleFragmentActivity{
@Override
protected Fragment createFragment(){
UUID crimeId = (UUID)getIntent().getSerializableExtra(CrimeFragment.EXTRA_CRIME_ID);
return CrimeFragment.newInstance(crimeId);
}
...
}
```
Fragment也有startActivityForResult(...)和onActivityResult(...)方法，但没有setResult(...)方法。

## 使用ViewPager展示Fragment
使用ViewPager配合PagerAdapter（实际是其子类FragmentPagerAdapter或FragmentStatePagerAdapter）来实现Fragment的切换，而不是使用AdapterView（AdapterView有一个和ViewPager类似的子类Gallery）与Adapter来实现，是因为：AdapterView无法使用现有的Fragment，需要编写代码及时地提供View，然而决定fragment视图何时创建的是FragmentManager，所以当Gallery要求Adapter提供fragment视图时，我们无法立即创建fragment并提供视图。PagerAdapter比Adapter复杂很多，因为其要处理更多的视图管理相关工作，其中代替使用getView()方法，PagerAdapter使用下列方法：
```java
public Object instantiateItem(ViewGroup container,int position);
public void destroyItem(ViewGroup container,int position,Object object);
public abstract boolean isViewFromObject(View view,Object object);
```

使用ViewPager时，只需为其设置PagerAdapter，并重写getCount()和getItem()这两个方法即可：
```java
public class CrimePagerActivity extends FragmentActivity{
private ViewPager mViewPager;
Private ArrayList<Crime> mCrimes;

@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate();
mViewPager = new ViewPager(this);
mViewPager.setId(R.id.viewPager);  // 
```
FragmentManager要求任何作为fragment的容器的视图都需要具有一个资源ID，可在res/values/下新建一个ids.xml，写入以下内容：
<resources xmlns:android=’...’>
<item type=”id” name=”viewPager”/>
</resources>
setContentView(mViewPager);  //  以代码方式定义视图，而不是传入一个布局的ID

mCrimes = CrimeLab.get(this).getCrimes();

FragmentManager fm = getSupportFragmentManager();
mViewPager.setAdapter(new FragmentStatePagerAdapter(fm){ //这里设置了和fm的关系，相当于委托它来负责具体的Fragment视图添加工作
```java
@Override
public int getCount(){ 
return mCrimes.size();
}

@Override
public Fragment getItem(int pos){ // 返回一个Fragment
Crime crime mCrime.get(pos);
return CrimeFragment.newInstance(crime.getId());  
}
});
}
}
```
除了FragmentStatePagerAdapter外，还有一个可用的PagerAdapter的子类FragmentPagerAdapter。两者的区别在于在卸载不再需要的fragment时，所采用的处理方法不同。FragmentStatePagerAdapter不会销毁掉不需要的fragment，在销毁fragment时，会将其onSaveInstanceState(Bundle)方法中的Bundle信息保存下来，用户切换回原来的页面后，保存的实例的状态可用于恢复生成新的fragment。而FragmentPagerAdapter对于不再需要的fragment则选择调用事务的detach(Fragment)方法，而非remove(Fragment)方法来处理，即只是销毁了fragment的视图，但仍将fragment实例保留在FragmentManager中，因此用FragmentPagerAdapter创建的fragment永远不会被销毁。通常来说，FragmentStatePagerAdapter更加节省内存，对于包含图片等内容比较大的fragment，最好选用FragmentStatePagerAdapter。
ViewPager默认加载当前屏幕上的列表项以及左右相邻页的数据，从而实现页面滑动的快速切换，可以通过调用setOffscreenPageLimit(int)方法来指定预加载相邻页面的数目。
ViewPager默认显示PageAdapter中的第一个列表项，可通过调用setCurrentItem()方法来设置具体项。

## 基于Fragment的对话框及同一个Activity的两个Fragment之间的数据传递
Android推荐的做法是将对话框，比如一个AlertDialog，封装在DialogFragment实例中，以通过FragmentManager来管理对话框，而不是直接显示。使用FragmentManager管理对话框可使用更多配置选项来显示对话框，另外如果设备发生旋转，独立配置使用的AlertDialog会在旋转后消失，而配置封装在fragment中的AlertDialog则不会有此问题。

要将DialogFragment添加给FragmentManager管理，并放到屏幕上，可调用以下两种方法：
public void show(FragmentManager manager, String tag)  // 事务可自动创建并提交
public void show(FragmentTransaction transaction, String tag)

从一个Fragment中打开另一个Fragment，并传值，使用Fragment Arguments。
从被打开的Fragment中返回数据，需要调用setTargetFragment()设置目标Fragment,并主动去调用目标Fragment的onActivityResult()方法，其效果类似Activity的sendResult()方法，但是Fragment没有该方法，因此可以自定义一个来实现同样的功能。

以在一个Fragment上点击弹出日期选择的对话框为例：
```java
// 定义一个DialogFragment的子类，重写其onCreateDialog()方法，返回一个对话框
public class DatePickerFragment extends DialogFragment{
@Override
public Dialog onCreateDialog(Bundle savedInstanceState){
View v = getActivity().getLayoutInflater().inflater(R.layout.dialog_date,null);
return new AlertDialog.Builder(getActivity())
.setView(v)  // 设置对话框的显示内容，如一个根节点为DatePicker的布局文件
.setTitle(‘...’)
	.setPositiveButton(
android.R.string.ok,
new DialogInterface.OnClickListener(){
public void onClick(DialogInterface dialog,int which){
sendResult(Activity.RESULT_OK);
}
}
)
.create();
}

// 模拟Activity的sendResult()方法
private void sendResult(int resultCode){  
if(getTargetFragment() == null)
return;

Intent i = new Intent();
i.putExtra(EXTRA_DATE,mDate);  // 设置返回值，mDate的值通过重写DatePicker的onDateChanged()方法进行设置
getTargetFragment().onActivityResult(getTargetRequestCode(),resultCode,i);
}
}

// 触发显示对话框
public class CrimeFragment extends Fragment{
...
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup parent,Bundle savedInstanceState){
mDateButton = (Button)findViewById(R.id.crime_date);
mDateButton.setOnClickListener(new View.OnClickListener(){
public void onClick(View v){
FragmentManager fm = getActivity().getSupportFragmentManager();
DatePickerFragment dialog = new DatePickerFragment();
dialog.setTargetFragment(CrimeFragment.this,REQUEST_DATE);
dialog.show(fm,DIALOG_DATE);
}
}); 
...
}

@Override
public void onActivityResult(int requestCode, int resultCode, Intent data){
// 接受、处理从Fragment中返回的数据
}
}
```
## 音频播放
```java
public class AudioPlayer{
private MediaPlayer mPlayer;

public void stop(){
if(mPlayer != null){
mPlayer.release(); // 除非调用release()方法，否则MediaPlayer将一直占着音频解码硬件及其他系统资源。另有一个stop()方法，会使MediaPlayer实例进入停止状态，需要时再重新启动。对于简单的音频播放应用，应该直接release()
mPlayer = null;
}
}

public void play(Context c){
stop(); // 避免用户多次单击Play按钮创建多个MediaPlayer实例
mPlayer = MediaPlayer.create(c, R.raw.one_small_step);  // R.raw.one_small_step是raw文件夹下的一个音频资源
// 监听音频播放结束事件，结束后主动调用stop()方法，释放占用资源
mPlayer.setCompletionListener(new MediaPlayer.OnCompletionListener(){
public void onCompletion(MediaPlayer mp){
stop();
}
});
mPlayer.start();
}
}
```
此外，因为MediaPlayer运行在一个不同的线程上，因此一旦启动，即使其所在Fragment被销毁了，MediaPlayer仍可以不停地播放。因此，需要覆盖Fragment的onDestroy()方法：
```java
	// 在Fragment销毁时，释放音频播放资源
@Override
public void onDestroy(){
super.onDestroy();
mPlayer.stop();
}
```

## Fragment的保留
调用setRetainInstance(true)。
保留Fragment利用了这样一个事实：可销毁和重建fragment的视图而无需销毁fragment自身。
fragment的restainInstance属性默认为false，表明其不会被保留。设置为true后可保留fragment，已保留的fragment不会随activity一起被销毁（如旋转设备），其全部实例变量值也将保持不变。当新的activity创建后，新的FragmentManager会找到被保留的Fragment，并重新创建它的视图。
虽然保留的fragment没有被销毁，但它已脱离消亡中的activity并处于保留状态，尽管此时fragment仍然存在，但已经没有任何activity在托管它。fragment必须同时满足两个条件才能进入保留状态：
1.已调用fragment的setRetainInstance(true)方法；
2.因设备配置改变，托管activity正在被销毁；
```java
@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
setRetainInstance(true);
}
```
Android并不鼓励保留Fragment，情况未知。

保留Fragment与重写onSaveInstanceState()方法的主要区别在于数据可以存多久。如只是短暂保存数据，则使用保留Fragment比较方便，可以不用操心要保留的对象是否实现了Serializable接口。但是如果需要持久化地存储数据，则需要重写onSaveInstanceState()方法，因为当用户短暂离开应用后，如果因为系统回收内存需要销毁activity，则保留的fragment也会被随之销毁。
 

## 配置修饰符
Android提供了用于不同语言的配置修饰符，因此简化了本地化处理的过程：首先创建带有目标语言配置修饰符的资源子目录，然后将可选资源放入其中。Android资源系统会为我们处理其他后续工作。可指定多重配置修饰符，各配置修饰符必须按照优先级顺序排列，例如values-zh-land是一个有效的资源目录名，但是values-land-zh则无效，因为语言配置符的优先级高于布局方向的优先级。
有些配置修饰符的兼容性并不具有非黑即白的排他性，例如API级别就不是一个严格匹配的修饰符，修饰符-v11可兼容运行API11级及更高级别的系统版本设备。
在同一个子目录下，不能以文件的扩展名为依据来区分命名相同的资源文件。
所有资源都必须保存在res/目录的子目录下，尝试在res/目录的根目录下保存资源将会导致编译错误。因为res/子目录的名字直接与Android的编译过程绑定。此外，也无法在res/的子目录下创建多级子目录。                                                                                                           

## 选项菜单
创建一个定义菜单的XML文件，放在res/menu目录下，Android会自动生成该XML文件的对应的资源ID：
<menu cmlns:android=’xxx’>
<item 
android:id=”@+id/menu_item_new_crime”
android:icon=”@android:drawable/ic_menu_add”
android:title=”@string/new_crime”
android:showAsAction=”ifRoom|withText”  // 其他可选值：always、never
/>
</menu>
showAsAction属性用于指定菜单选项是显示在操作栏上，还是隐藏到溢出菜单中。

在Fragment中实例化选项菜单：
```java
public class CrimeListFragment extends ListFragment{
...
@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
setHasOptionsMenu(true);  // Fragment的onCreateOptionMenu(...)方法是由FragmentManager负责调用的，因此当activity接收到来自操作系统的onCreateOptionMenu(...)方法调用时，必须明确告诉FragmentManager：其所管理的fragment应接收onCreatOptionMenu(...)方法的调用
...
}

@Override
public void onCreateOptionMenu(Menu menu, MenuInflater inflater){
super.onCreateOptionsMenu(menu,inflater);
inflater.inflate(R.menu.fragment_crime_list,menu); // 加载菜单布局
}

@Override
public boolean onOptionsItemSelected(MenuItem item){ // 响应菜单点击事件
switch(item.getItemId()){
case R.id.menu_item_new_crime:
...
return true;
default:
return super.onOptionsItemSelected(item);
}
}

}
```
## 使用上下文操作实现列表视图的多选操作
上下文操作是一种与某个特定屏幕视图而非整个屏幕相关联的操作，通过长按视图触发，例如删除列表中的某一项。在API11以下的设备上展示为一个浮动菜单，在较新的设备上通过上下文操作栏呈现。
在较新设备上长按视图进入上下文操作模式是提供上下文操作的主流方式，屏幕进入上下文操作模式时，上下文菜单中定义的菜单项出现并覆盖操作栏。相比浮动菜单，上下文操作栏不会遮挡屏幕，因而是更好的菜单展现方式。
列表视图进入上下文操作模式时，可开启它的多选模式。多选模式下，上下文操作栏上的任何操作都将同时应用于所有已选视图。
```java
public class CrimeListFragment extends ListFragment{
@TargetApi(11)
@Override
public View onCreateView(LayoutInflater inflater,ViewGroup parent,Bundle savedInstanceState){
View v = super.onCreateView(inflater,parent,savedInstanceState);
ListView listView = (ListView)v.findViewById(android.R.id.list);

// 设置列表视图的选择模式
listView.setChoiceMode(ListView.CHOICE_MODE_MULTIPLE_MODAL);  

// MutiChoiceModeListener实现了另一个接口：ActionMode.Callback，用户屏幕进入上下文操作模式时，会创建一个ActionMode实例，之后在其生命周期内Callback接口的回调方法会在不同时点被调用
listView.setMultiChoiceModelListener(new MultiChoiceModeListener(){ 

// 视图在选中，或撤销选中时触发
public void onItemCheckedStateChanged( 
ActionMode mode, int position, long id, boolean checked){
...
}

// 在ActionMode对象创建后调用，也是实例化上下文菜单资源，并显示在上下文操作栏上的任务完成的地方
public boolean onCreateActionMode(ActionMode mode, Menu menu){
// 注意这里是从ActionMode对象获取MenuInflater对象，而不是通过Activity对象来获得。ActionMode负责对上下文操作栏进行配置，比如设置标题等，Activity的MenuInflater做不到这一点。
MenuInflater inflater = mode.getMenuInflater();
inflater.inflate(R.menu.crime_list_item_context,menu); // 渲染菜单视图
return true;
}

// 在onCreateActionMode()方法之后，以及当前上下文操作栏需要刷新显示新数据时调用
public boolean onPrepareActionMode(ActionMode mode,Menu menu){
return false;
}

// 在用户选中某个菜单项操作时调用，是响应上下文菜单项操作的地方
public boolean onActionItemClicked(ActionMode mode,MenuItem item){
switch(item.getItemId()){
case R.id.menu_item_delete_crime:
CrimeAdapter adapter = (CrimeAdapter)getListAdapter();
CrimeLab crimeLab = CrimeLab.get(getActivity());
for(int i = adapter.getCount()-1; i>=0; i--){
// 多选模式，删除一个或多个Crime对象
if(getListView().isItemChecked(i)){
crimeLab.deleteCrime(adapter.getItem(i));
}
}
mode.finish();
adapter.notifyDataSetChanged();
return true;
default:
return false;
}
}

// 在用户退出上下文操作模式或所选菜单项操作已被响应，从而导致ActionMode对象将要销毁时调用
public void onDestroyActionMode(ActionMode mode){
..
}
});
return v;
}
```
以上通过使用MultiChioceModeListener接口的方式，会自动创建ActionMode实例。因此以上解决方案可应用于ListView或GridView。对于其他视图，要使用上下文操作栏，需要实现一个View.OnLongClickListener接口的监听器，然后在监听器实现中调用Activity.startActionMode(...)方法创建一个ActionMode实例。startActionMode(...)方法需要一个实现了ActionMode.Callback接口的对象作为参数，而该接口即包含了上述的各个回调方法。
## 层级式导航
使用后退键的导航称为临时性导航，只能返回到上一次的用户界面。层级式导航可逐级向上在应用内导航。Android可轻松利用操作栏上的应用图标实现层级式导航，即逐级向上直至应用的初始界面。通常，应用图标一旦启用了向上导航按钮的功能，在应用图标的左边就会显示一个向左指向的图标。

在定义应用图标的点击响应事件时，回到某Activity可以使用Intent.FLAG_ACTIVITY_CLEAR_TOP来启动指定的Activity。然而，Android有更好的办法实现层级式导航：配合使用NavUtils以及manifest中的元数据。
首先修改AndroidManifest.xml文件，在CrimePagerActivity声明中添加meta-data属性，指定CrimePagerAactivity的父类为CrimeListActivity：
<activity 
android:name=”.CrimePagerActivity”
android:label=”@string/app_name”>
<meta-data
android:name=”android.support.PARENT_ACTIVITY”
android:value=”.CrimeListActivity”>
</activity>

```java
public class CrimeFragment extends Fragment{
...
@Override
public void onCreate(Bundle savedInstanceState){
...
setHasOptionsMenu(true);  
}

// 响应应用图标菜单项
@Override
public boolean onOptionsItemSelected(MenuItem item){ // 响应菜单点击事件
switch(item.getItemId()){
case android.R.id.home:  // 响应回到父Activity
if(NavUtils.getParentActivityName(getActivity())!=null){
NavUtils.navigateUpFromSameTask(getActivity());
}
return true;
default:
return super.onOptionsItemSelected(item);
}
}


@TargetApi(11)
@Override
public View onCreateView(LayoutInflater inflater,ViewGroup parent,Bundle savedInstanceState){
...
if(BUILD.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB){
getActivity().getActionBar.setDisplayHomeAsUpEnabled(true);  // 启用应用图标的向上导航功能，但这仅仅是让应用图标转变为按钮，显示一个向左的图标而已
}
...
}
```
## 操作json
定义一个支持json转换的类：
```java
public class Crime{
private static final String JSON_ID = “id”;
private static final String JSON_TITLE = “title”;
private static final String JSON_DATE = “date”;

private UUID mID;
private String mTitle;
private Date mDate = new Date();

public Crime(){
mId = UUID.randomUUID();
}

// json对象转换为属性
public Crime(JSONObject json) throws JSONException{
mId = UUID.fromString(json.getString(JSON_ID));
if(json.has(JSON_TITLE)){
mTitle = json.getString(JSON_TITLE);
}
mDate = new Date(json.getLong(JSON_DATE));
}

// 属性转换成json对象
public JSONObject toJSON() throws JSONException{
JSONObject json = new JSONObject();
json.put(JSON_ID,mId.toString());
json.put(JSON_TITLE,mTitle);
json.put(JSON_DATE,mDate.getTime());
return json;
}
}
```
基于支持json转换的类，实现json文件互转的类：
```java
public class CriminalIntentJSONSerializer{
private Context mContext;
private String mFilename;
public CriminalIntentJSONSerializer(Context c, String f){
mContext = c;
mFilename = f;
}

// 将类实例对象的数组转换为JSONObject的数组，再调用JSONObject的toString()方法转换为字符串，然后写入文件中
public void saveCrimes(ArrayList<Crime> crimes) throws JSONException,IOException{
JSONArray array = new JSONArray();
for(Crime c:crimes)
array.put(c.toJSON());
Writer writer = null;
try{
OutputStream out = mContext.openFileOutput(mFilename,Context.MODE_PRIVATE);
writer = new OutputStreamWriter(out);
writer.write(array.toString());
}
finally{
if(writer != null)
writer.close();
}
}

// 读取json文件，拼接为字符串，解析为JSONArray，再转换为所需的类的实例
public ArrayList<Crime> loadCrimes() throws JSONException,IOException{
ArrayList<Crime> crimes = new ArrayList<Crime>();
BufferedReader reader = null;
try{
InputStream in = mContext.openFileInput(mFilename);
reader = new BufferdReader(new InputStreamReader(in));
StringBuilder jsonString = new StringBuilder();
String line = null;
while((line = reader.readline())!=null){
jsonString.append(line);
}
JSONArray array = (JSONArray)new JSONTokener(jsonString.toString()).nextValue(); // JSONTokener是json文本解析类
for(int i=0; i<array.length(); i++){
crimes.add(new Crime(array.getJSONObject(i)));
}
}
catch(FileNotFoundException e){
...
}
finally{
if(reader != null)
reader.close();
}
retuen crimes;
}
}
```
## 相机取景
可以通过隐式Intent来与照相机进行交互，相机应用会自动侦听MediaStore.ACTION_IMAGE_CAPTURE创建的Intent。
 ![image](https://github.com/woojean/woojean.github.io/blob/master/images/android_4.png)
Surface对象代表原始像素数据的缓冲区。SurfaceView类实现了SurfaceHolder接口，通过该接口来操作Surface。SurfaceView出现在屏幕上时，会创建Surface对象，当SurfaceView从屏幕上消失时，Surface对象即被销毁。不同于其他对象，SurfaceView不会自我绘制内容，任何想将内容绘制到Surface缓冲区的对象，都称为Surface的客户端，如Camera。SurfaceHolder提供了一个接口SurfaceHolder.Callback，该接口监听Surface生命周期中的事件，以此控制Surface与其客户端协同工作。

创建一个Fragment布局，包含一个SufaceView、一个Button和一个进度条ProgressBar：
<FrameLayout>
<LinearLayout>
<SurfaceView />
<Button />
<FrameLayout
android:id=”@+id/crime_camera_progressContainer”
android:layout_width=”match_parent”
android:layout_height=”match_parent”
android:clickable=”true”  //  截获点击、触摸事件，这样，当该FrameLayout显示时，可以阻止用户与LinearLayout组件包含的子组件交互，如点击拍照
/>
<ProgressBar
style=”@android:style/Widget.ProgressBar.Large”
...
/>

创建Fragment：
```java
public class CrimeCameraFragment extends Fragment{
private Camera mCamera; 
private SurfaceView mSufaceView;
private View mProgressContainer;

// 在相机捕获图像时调用，显示进度条
private Camera.ShutterCallback mShutterCallback = new Camera.ShutterCallback(){
public void onShutter(){
mProgressContainer.setVisibility(View.VISIBLE);
}
}

// 在JPEG格式的图像可用时调用：保存图像为文件
private Camera.PictureCallback mJpegCallback = new Camera.PictureCallback(){
public void onPictureTaken(byte[] data, Camera camera){
		String filename = UUID.randomUUID().toString()+“.jpg”；
FileOutputStream os = null;
boolean success = true;
try{
os = getActivity().openFileOutput(filename,Context.MODE_PRIVATE);
os.write(data);
}
catch(Exception e){
success = false;
}
finally{
try{
if(os!=null)
os.close();
}catch(Exception e){
success = false;
}
}
if(success){
...
}
getActivity().finish();
}

@Override
@SuppressWarning(“deprecation”)
public View onCreateView(LayoutInflater infalter,ViewGroup parent,Bundle savedInstanceState){
View v = inflater.inflate(R.layout.fragment_crime_camera,parent,false);
mProgressContainer = v.findViewById(R.id.crime_camera_progressContainer);
mProgressContainer.setVisibility(View.INVISIBLE);  //  进度条初始不可见
Button takePictureButton = (Button)v.findViewById(R.is.crime_camera_takePictureButton);

takePictureButton.setOnclickListener(new View.OnClickListener(){
public void onClick(View v){
if(mCamera != null){
// Camera的takePicture方法：
public final void takePicture(
Camera.ShutterCallback shutter,  在相机捕获图像时调用
Camera.PictureCallback raw,  在原始图像数据可用时调用
Camera.PictureCallback jpeg  在JPEG版本的图像可用时调用
)
mCamera.takePicture(mShutterCallback,null,mJpegCallback); // 设置相机回调方法
}
});
mSurfaceView = (SurfaceView)v.findViewById(R.id.crime_camera_surfaceView);
SurfaceHolder holder = mSurfaceView.getHolder();
holder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);

// 实现SurfaceHolder.Callback接口，使得相机预览与Surface生命周期方法能够协同工作
holder.addCallBack(new SurfaceHolder.Callback(){
public void surfaceCreated(SurfaceHolder holder){
try{
if(mCamera != null ){
mCamera.setPreviewDisplay(holder); // 连接camera与surface
}
}
catch(IOException exception){
...
}
}

public void surfaceDestroyed(SurfaceHolder holder){
if(mCamera != null ){
mCamera.stopPreview();// 停止在surface上绘制
}
}

// surface首次显示在屏幕上时调用
public void surfaceChanged(SurfaceHolder holder,int format,int w,int h){
if(mCamera == null ) return;
Camera.Parameters parameters = mCamera.getParmeters();
Size s = ...; // 计算最合适尺寸的逻辑略去
parameters.setPreviewSize(s.width,s.height);
try{
mCamera.startPreview();  // 在surface上绘制帧
}
catch(Exception exception){
mCamera.release();
mCamera = null;
}
}

return v;
}

@Target(9)
@Override
public void onResume(){ // 在onResume中打开相机
if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD){
mCamera = Camera.open(0); // 0打开后置相机，1打开前置相机
}
else{
mCamera = Camera.open(); // API9之前的设备不支持带int参数的open方法
}
}

@Override
public void onPause(){ // 在onPause中释放相机资源
super.onPause();
if(mCamera != null){
mCamera.release();
mCamear = null;
}
}

}
```
创建一个托管该Fragment的Activity：
```java
public class CrimeCameraActivity extends SingleFragmentActivity{
@Override
public void onCreate(Bundle savedInstanceState){
// 隐藏操作栏和状态栏的操作只能在Activity中实现，而不能再Fragment中实现，因为它们必须在setContentView()方法调用之前调用。当然这里的Activity根本没有调用setContentView()方法，它是一个SingleFragmentActivity，通过覆盖createFragment()方法来设置自己视图
requestWindowFeature(window.FEATURE_NO_TITLE);  // 隐藏状态栏
getWindow().addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);  // 全屏
super.onCreate(savedInstanceState);

@Override 
protected Fragment createFragment(){
return new CrimeCameraFragment();
}
}
```
修改Manifest文件，增加新的Activity的定义，以及使用相机的权限：
<manifest
...
<uses-permission android:name=”android.permission.CAMERA” />  //  需要相机权限
<uses-feature android:name=”android.hardware.camera” />  //  只有带相机的设备才能使用该APP

<application
...
<activity
android:name=”.CrimeCameraActivity”
android:screenOrientation=”landscape”  //  强制始终横屏显示
android:label=”@string/app_name”>
</activity>

## 隐式Intent
使用隐式Intent可以启动其他应用的Activity。使用显式Intent，需要明确指定要启动的Activity类名，而使用隐式Intent只需要向操作系统描述清楚工作意图，操作系统会去启动那些对外宣传能够胜任工作的Activity。一个隐式Intent主要包含以下部分：
（1）要执行的操作；
（2）要访问的数据的位置，如网页URL、指向文件的URI、指向ContentProvider中记录的URI；
（3）操作涉及的数据的类型，即MIME形式的数据类型；
（4）可选类别：用来描述如何使用某个Activity。
如下，通过<intent-filter设置，该Activity可以对外宣称自己是适合处理ACTION_VIEW的activity：
<activity
android:name=”.BrowserActivity”
android:label=”@string/app_name>
<intent-filter>
<action android:name=”android.intent.action.VIEW” />
<category android:name=”android.intent.category.DEFAULT” />
<data android:scheme=”http” android:host=”www.xxx.com” />
</intent-filter>
</activity>
DEFAULT类别必须明确地在intent过滤器中进行设置。DEFAULT类别实际隐含添加到了几乎每一个隐式intent中（所以要想接收到这种隐式intent，就必须显示声明该类别），唯一的例外是LAUNCHER类别。
隐式intent也可以包含extra信息，不过操作系统在寻找合适的activity时不会使用extra信息。

例：一个指定发送一段文本信息的隐式intent（文本信息既可以通过短信应用发送，也可以通过邮件等其他应用发送，它们都可以处理如下声明的intent）
```java
Intent i = new Intent(Intent.ACTION_SEND);
i.setType(“text/plain”);
i.putExtra(Intent.EXTRA_TEXT,getCrimeReport());
i.putExtra(Intent.EXTRA_SUBJECT,getString(R.string.crime_report_subject));
i = Intent.createChooser(i,getString(R.string_send_report));  // 始终创建一个供选择的Activity，即使只有一个符合条件的应用
startActivity(i);
```

例：获取联系人信息
```java
// 发送隐式Intent
Intent i = new Intent(Intent.ACTION_PICK, // 动作
ContactsContract.Contacts.CONTENT_URI); // 数据位置
startActivityForResult(i,REQUEST_CONTACT);

// 使用ContentResolver解析ContentProvider的返回值
@Override
public void onActivityResult(int requestCode,int resultCode,Intent data){
...
if(requestCode == REQUEST_CONTACT){
Uri contactUri = data.getData();
String[] queryFields = new String[]{
ContactsContract.Contacts.DISPLAY_NAME;
}
Cursor c = 		getActivity().getContentResolver().query(contactUri,queryFields,null,null,null);
if(c.getCount() == 0){
c.close();
return;
}
c.moveToFirst();
String suspect = c.getString(0)；
mCrime.setSuspect(suspect);
c.close();
}
}
```
可用Activity的检查：如果操作系统找不到匹配的Activity，就会崩溃。一种处理方法是在onCreateView()方法中对可用Activity进行检查：
```java
PackageManager pm = getPackageManager();
List<ResolverInfo> activities = pm.queryIntentActivities(i,0);
boolean isIntentSafe = activities.size() > 0;
```

## 根据设备的尺寸使用合适的布局
实现双版面布局：在手机设备上实例化单版面布局，在平板设备上实例化双版面布局。
创建单版面布局：layout/activity_fragment.xml
创建双版面布局：layout/activity_twopane.xml
创建默认的别名资源值，指向单版面布局：res/values/refs.xml
<resources>
<item name=”activity_masterdetail” type=”layout”>@layout/activity_fragment</item>
</resources>
创建用于大屏幕设备的别名资源值，指向双版面布局：res/values-sw600dp/refs.xml
<resources>
<item name=”activity_masterdetail” type=”layout”>@layout/activity_twopane</item>
</resources>
注意：以上两个别名名称相同，但位于不同的目录下。sw的意思是smallest width，sw600dp，即最小宽度600dp，实际指设备屏幕的最小尺寸。
定义一个方法，用来返回布局id：
```java
protected int getLayoutResId(){
return R.layout.activity_masterdetail;
}
```
更新Activity的setContentView()的参数：
```java
@Override
public void onCreate(Bundle savedInstanceState){
...
setContentView(getLayoutResId());
```

## 基于Fragment的Master-Detail用户界面
为了保持fragment的独立性，可以在fragment中定义回调接口，委托托管它的activity来完成那些不应该由fragment处理的任务。托管activity将实现回调接口，履行托管fragment的任务。有了回调接口，fragment可以直接调用托管activity的方法，而无需知道自己的托管者是谁。

具体到本例，就是为CrimeListFragment添加回调接口，然后在托管它的Activity中实现这个接口。这样，当CrimeListFragment中的某个列表项被点击时，可以调用其托管Activity添加相应的CrimeFragment（即添加Detail信息）。

为CrimeListFragment类添加回调接口：
```java
public class CrimeListFragment extends ListFragment{
private ArrayList<Crime> mCrime;
private boolean mSubtitleVisible;
private Callbacks mCallbacks;

public interface Callbacks{  // 定义一个回调接口
void onCrimeSelected(Crime crime);
}

@Override
public void onAttach(Activity activity){
super.onAttach(activity);
mCallbacks = (Callbacks)activity; // 因为托管Activity实现了该接口，所以可以转型
}

@Override
public void onDetach(){
super.onDetach();
mCallbacks = null;
}

public void onListItemClick(ListView l,View v,int position,long id){
Crime c = ((CrimeAdapter)getListAdapter()).getItem(position);
mCallbacks.onCrimeSelected(c);  // 触发回调
}
...
}

Activity实现回调接口
public class CrimeListActivity extends SingleFragmentActivity
implements CrimeListFragment.Callbacks{

@Override
protected Fragment createFragment(){
return new CrimeListFragment();
}

@Override
protected int getLayoutResId(){
return R.layout.activity_masterdetail;
}

public void onCrimeSelected(Crime crime){  //  实现接口方法
if(findViewById(R.id.detailFragmentContainer) == null){ // 手机布局
Intent i = new Intent(this,CrimePagerActivity.class);
i.putExtra(CrimeFragment.EXTRA_CRIME_ID,crime.getId());
startActivity(i);
}
else{ // 平板布局
FragmentManager fm = getSupportFragmentManager();
FragmentTransaction ft = fm.beginTransaction();

Fragment oldDetail = fm.findFragmentById(R.id.detailFragmentContainer);
Fragment newDetail = CrimeFragment.newInstance(crime.getId());
if(oldDetail!=null){
ft.remove(oldDetail); // 去掉旧的Detail
}
ft.add(R.id.detailFragmentContainer,newDetail);  // 添加新的Detail
ft.commit();
}
}
}
```
同理，如果想实现在Detail中修改信息，比如标题，提交后列表项也响应更新，只需在CrimeFragment中也定义一个内部接口，然后Activity同时再实现这个接口。当CrimeFragment中的更新操作发生时，主动通过mCallbacks对象触发Activity的UI更新操作（notifyDataSetChanged）。
总而言之，这是一种模式：父对象想要根据子对象的变化来更新自己，则需要在子对象中定义回调接口，父对象实现回调接口。子对象通过某种方式可以获取到父对象的引用，并把它转型为回调接口。当子对象的变化发生时，主动通过回调接口对象来触发父对象的相应的变化。

## 创建替换Android默认启动器的应用
该应用基于这样一个基本事实：所有应用的主Activity都会响应这样一个隐式Intent，该隐式Intent包括一个MAIN操作和一个LAUNCHER类别：
<intent-filter>
<action android:name=”android.intent.action.MAIN” />
<category android:name=”android.intent.category.LAUNCHER” />
</intent-filter>

```java
public class NerdLauncherFragment extends ListFragment{
@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
// 定义隐式Intent
Intent startupIntent = new Intent(Intent.ACTION_MAIN);
startupIntent.addCategory(Intent.CATEGORY_LAUNCHER);

// 获取所有应用的主Activity的集合
PackageManager pm = getActivity().getPackageManager();
List<ResolverInfo> activities = pm.queryIntentActivities(startupIntent,0);

// 对主Activity进行排序
Collections.sort(activities,new Comparator<ResolveInfo>(){
public int compare(ResolveInfo a,ResolveInfo b){
PackageManager pm = getActivity().getPackageManager();
return String.CASE_INSENSITIVE_ORDER.compare(
a.loadLabel(pm).toString(),b.loadLabel(pm).toString());
}
}

// 封装Activity集合，得到一个列表的数据适配器
ArrayAdapter<ResolveInfo> adapter = new ArrayAdapter<ResolveInfo>(
getActivity(),android.R.layout.simple_list_item_1,activities){
public View getView(int pos,View convertView,ViewGroup parent){
PackageManager pm = getActivity().getPackageManager();
View v = super.getView(pos,convertView,parent);
TextView tv = (TextView)v;
ResolveInfo ri = getItem(pos);
tv.setText(ri.loadLabel(pm));
return v
}
}
setListAdapter(adapter);
}

@Override
public void onListItemClick(ListView l,View v,int position,long id){
ResolvedInfo resolveInfo = (ResolveInfo)l.getAdapter().getItem(position);
ActivityInfo activityInfo = resolveInfo.activityInfo;
if(activityInfo == null) return;

// 可见，显式Intent也可以指定动作
Intent i = new Intent(Intent.ACTION_MAIN);
i.setClassName(activityInfo.applicationInfo.packageName,activityInfo.name);
startActivity(i);
}
```
MAIN/LAUNCHER intet过滤器可能无法与通过startActivity(...)方法发送的MAIN/LAUNCHER隐式intent相匹配。因为调用startActivity(Intent)方法意味着启动与发送的Intent相匹配的默认的activity，操作系统会默认将Intent.CATEGORY_DEFAULT类别添加给目标intent。因此，如果希望一个intent过滤器能够与通过startActivity(...)发送的隐式intent相匹配，那么必须在对应的intent过滤器中包含DEFAULT类别。而应用的主Activity通常不包含DEFAULT类别。

将该应用设为设备主屏幕：
<intent-filter>
<action android:name=”android.intent.action.MAIN” />
<category android:name=”android.intent.category.LAUNCHER” />
<category android:name=”android.intent.category.HOME” />
<category android:name=”android.intent.category.DEFAULT” />
</intent-filter>
点击Home键，该应用会成为可选的主界面。
如果要恢复到系统默认设置，可以选中Settings->Applications->Manage Application菜单项，找到该应用，并清除它的Launch by default选项。

## 任务、后退栈、进程
任务是用户比较关心的Activity栈。用户点击后退键时，栈顶activity会弹出栈外，如果当前屏幕上显式的是基activity，点击后退键，系统将退回主屏幕。
默认情况下新activity都在当前任务中启动，即使要启动的activity不属于CriminalIntent应用，它同样也是在当前任务中启动（不同应用的activity可以属于同一个任务），在当前任务中启动activity的好处是用户可以在任务内而不是应用层级间导航返回。
要想在新任务中启动Activity，需要给Intent添加一个标志：
i.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
这样，新启动的Activity在任务栏中会显示为独立的一项，而不是显示在当前任务中。

进程是操作系统创建的供应用对象生存以及应用运行的地方。在Android系统中，进程总会有一个运行的Dalvik虚拟机。
进程和任务是两个容易混淆的概念：activity赖以生存的任务和进程可能会有所不同，例如在CriminalIntent应用中启动联系人应用选择联系人时，虽然联系人activity是在CriminalIntent任务中启动的，但它是在联系人应用的进程中运行的。这也意味着，用户点击后退键在不同的activity间导航时，他可能还没有意识到正在进程间切换。
不能终止任务，但是可以终止进程。Google Play中那些宣称自己是任务终止器的应用实际上都是进程终止器。

## HTTP与后台任务
Android禁止在主线程中发生任何网络连接行为，强行为之，会抛出NetworkOnMainThreadException。

定义GridView每一项的布局：
<ImageView
android:id=”@+id/gallery_item_imageView”
android:layout_width=”match_parent”
android:layout_height=”120dp”
android:layout_gravity=”center”
android:scaleType=”centerCrop”  // 居中放置图片，然后进行放大，即放大较小图片、裁剪较大图片以匹配视图
/>

定义将要加载的模型对象类：
```java
	public class GalleryItem{
private String mCaption;
private String mId;
private String mUrl;

public String toString(){
return mCaption;
}
}
```

定义一个基本的HTTP访问类：
```java
public class FlickrFentchr{

private static final String ENDPOINT = “http://api.flickr.com/...”;
private static final String API_KEY= “...”;
private static final String METHOD_GET_RECENT= “...”;
 
private static final String PARAM_EXTRAS= “...”;
private static final String EXTRA_SMALL_URL= “...”;

private static final String XML_PHOTO= “photo”;

// byte数组转String
public String getUrl(String urlSpec) throws IOException{
return new String(getUrlBytes(urlSpec)); 
}

// 打开HTTP链接，将结果读入byte数组中
private byte[] getUrlBytes(String urlSpec) throws IOException{
URL url = new URL(urlSpec);
HttpURLConnection connection = (HttpURLConnection)url.openConnection();
try{
ByteArrayOutputStream out = new ByteArrayOutputStream(); // 输出流，向byte数组写数据
InputStream in = connection.getInputStream();

if(connection.getResponseCode() != HttpURLConnection.HTTP_OK){
return null;
}
int bytesRead = 0;
byte[] buffer = new byte[1024];
while((byteRead = in.read(buffer))> 0){
out.write(buffer,0,bytesRead);
}
out.close();
return out.toByteArray();
}finally{
connection.disconnect();
}
}

// 拼凑URL，调用请求方法
public void fetchItems(){
try{
String url = Uri.parse(ENDPOINT).buildUpon()
.appendQueryParameter(“method”,METHOD_GET_RECENT)
.appendQueryParameter(“api_key”,API_KEY)
.appendQueryParameter(PARAM_EXTRAS,EXTRA_SMALL_URL)
.build().toString();

String xmlString = getUrl(url);
}catch(IOException ioe){
...
}
}

// 解析得到的XML，得到模型对象的数组
private void parseItems(ArrayList<GalleryItem> items,XmlPullParser parser) 
throws XmlPullParserException,IOException{
int eventType = parser.next();
while(eventType != XmlPullParser.END_DOCUMENT){
if(eventType == XmlPullParser.START_TAG && XML_PHOTO.equals(parser.getName())){
String id = parser.getAttributeValue(null,”id”);
String caption = parser.getAttributeValue(null,”title”);
String smallUrl= parser.getAttributeValue(null,EXTRA_SMALL_URL);

GalleryItem item = new GalleryItem();
item.setId(id);
item.setCaption(caption);
item.setUrl(smallUrl);
items.add(item);
}
eventType = parser.next();
}
}
}
```
定义后台线程类
Android中使用消息队列的线程叫做消息循环（Message Loop)。消息循环会不断循环检查队列上是否有新消息。消息循环由一个线程和一个looper组成，Looper对象管理着线程的消息队列。
在Looper.loop()方法中执行了white(true)循环，在循环体内执行了Message msg = queue.next()，即在循环体内操作了消息队列。
主线程也是一个消息循环，因此也有一个looper。在哪个线程上创建的Handler，就与该线程的Looper相关联。
一般的Thread显然不带looper，因此需要自己调用Looper.prepare()和Looper.loop()，以及Handler。
HandlerThread为自带Looper的Thread类。
Handler可以post一个runnable，也可以send一个message。post一个runnable实际上仍然是通过sendMessage来实现的，因此本质上只有sendMessage一种方式。

Message包含好几个实例变量，其中有3个需要在实现时定义：
what:用户定义的int型消息代码
obj:随消息发送的用户指定的对象
target:处理消息的Handler，因此Message在创建时总是与一个Handler相关联
Handler不仅是处理Message的目标，也是创建和发布Message的接口。


以下ThumbnailDownloader这个类配合Fragment中的GridView实现图片的按需加载。所谓按需，即在GridView的adapter的getView()中触发下载行为。
```java
public class ThumbnailDownloader<Token> extends HandlerThread{
private static final int MESSAGE_DOWNLOAD = 0;

Handler mHandler; // 这个handler用来操纵自身所在线程的消息队列

// 一个同步map，用来存储和获取与特定Token相关的URL
Map<Token,String> requestMap =
Collections.synchronizedMap(new HashMap<Token,String>()); 

Handler mResponseHandler;  // 来自启动线程的Handler，在构造方法中赋值
Listener<Token> mListener; // 来自启动线程的Listener，用于在本线程中执行任务后回调

public interface Listener<Token>{
void onThumbnailDownloaded(Token token,Bitmap thumbnail);
}

public void setListener(Listener<Token> listener){
mListener = listener;
}

public ThumbnailDownloader(Handler responseHandler){
mResponseHandler = responseHandler;
}


// 定义handler如何处理消息
@SuppressLint(“HandlerLeak”)
@Override
protected void onLooerPrepared(){
mHandler = new Handler(){
@Override
public void handeMessage(Message msg){
if(msg.what == MESSAGE_DOWNLOAD){
@SuppressLint(“unchecked”)
Token token = (Token)msg.obj; // 由于类型擦除，因此这里的强制类型转换应该是不允许的，所以要添加@SuppressLint(“unchecked”)
handleRequest(token);
}	
}
}
}

private void handleRequest(final Token token){
try{
final String url = requestMap.get(token);
if(url == null) return;

byte[] bitmapBytes = new FlickrFetchr().getUrlBytes(url); 
final Bitmap bitmap = 				 
BitmapFactory.decodeByteArray(bitmapBytes,0,bitmapBytes.length);

// mResponseHandler和mListener都来自启动线程，即主线程
mResponseHandler.post(new Runnable(){
public void run(){
if(requestMap.get(token) != url) return;
requestMap.remove(token);
mListener.onThumbnailDownloaded(token,bitmap);  // 更新图片
}
}
}catch(IOException ioe){
...
}
}

// 清理
public void clearQueue(){
mHandler.removeMessages(MESSAGE_DOWNLOAD);
requestMap.clear();
}

// 该方法在启动线程中调用（在getView()中调用）
public void queueThumbnail(Token token,String url){
requestMap.put(token,url);

// 从公共循环池中获取一个Message对象，组装后，发送
mHandler.obtainMessage(MESSAGE_DOWNLOAD,token).sendToTarget();
}

}


定义用来加载图片的Fragment
public class PhotoGalleryFragment extends Fragment{
GridView mGridView;
ArrayList<GalleryItem> mItems;
ThumbnailDownloader<ImageView> mThumbnailThread;


@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
setRetainInstance(true);
new FetchItemTask().fetchItems();  // 执行异步任务

// 主线程将自己的一个handler传递给子线程
mThumbnailThread = new ThumbnailDownloader<ImageView>(new Handler());

// 主线程将自己的回调接口对象传递给子线程
mThumbnailThread.setListener(new ThumbnailDownloader.Listener<ImageView>(){
public void onThumbnailDownloaded(ImageView imageView,Bitmap thumbnail){
if(isvisiable()){
imageView.setImageBitmap(thumbnail);
}
}
});
mThumbnailThread.start();
mThumbnailThread.getLooper();
}

@Override
public void onDestroy(){
super.onDestroy();
mThumbnailThread.quit(); 
}

@Override
public View onCreateView(LayoutInflater inflater,
ViewGroup container,Bundle savedInstanceState){
View v = inflater.inflate(R.layout.fragment_photo_gallery,container,false);
mGridView = (GridView)v.findViewById(R.id.gridView);

setupAdapter();

return v;
}

void setupAdapter(){
if(getActivity() == null || mGridView == null) return;
if(mItems != null){
// 这里没有定义ArrayAdapter的getView()方法，将调用toString(),展示文本
mGridView.setAdapter(new ArrayAdapter<GalleryItem>(
getActivity(),
android.R.layout.simple_gallery_item,
mItems));

mGridView.setAdapter(new GalleryItemAdapter(mItems));
}
else{
		mGridView.setAdapter(null);
}
}

// 实现GridView显示图片的Adapter
private class GalleryItemAdapter extends ArrayAdapter<GalleryItem>{
public GalleryItemAdapter(ArrayList<GalleryItem> items){
super(getActivity(),0,items);
)

// 覆盖getView方法
@Override
public View getView(int position,View convertView, ViewGroup parent){
if(convertView == null){
convertView = 
getActivity()
.getLayoutInflater()
.inflate(R.layout.gallery_item,parent,false);
}
// 在返回convertView之前,拿到其内部的ImageView的引用，传递给任务类
ImageView imageView = 			
		(ImageView)convertView.findViewById(R.id.gallery_item_imageView);
ImageView.setImageResource(R.drawable.xxx); // 设置一个默认图片，之后再动态替换

// 在getView中实现“按需”下载图片
GalleryItem item = getItem(position);

// 传递实体对象，通知任务类，任务类将基于实体对象发出消息
mThumbnailThread.queueThumbnail(imageView,item.getUrl()); 

return convertView;
}

// 基于AsyncTask，实现一个内部的异步任务工具类
private class FetchItemTask extends AsyncTask<void,void,ArrayList<GalleryItem>>{
@Override
protected void doInBackground(void...params){
try{
return new FlickrFetchr().fetchItems();
}catch(IOException ioe){
...
}
return null;
}

// doInBackground的返回值就是onPostExcute的输入值
onPostExcute方法运行在主线程上，且在doInBackground方法运行后执行，因此可以用来更新UI
@Override
protected void onPostExcute(ArrayList<GalleryItem> items){
mItems = items;
setupAdapter();
}
}

@Override
public void onDestroyView(){
super.onDestroyView();
mThumbnailThread.clearQueue();
}
}

```
定义加载图片Fragment的托管Activity
```java
public class PhotoGalleryActivity extends SingleFragmentActivity{
@Override
public Fragment createFragment(){
return new PhotoGalleryFragment();
}
}
```
修改Manifest，获取网络使用权限
<manifest xmlns:android=”...”
...
<uses-permission android:name=”android.permission.INTERNET” />



## 用AsyncTask更新进度条
功能实现主要基于这样的现实：AsyncTask类有一个运行在单独线程中的publishProgress方法和配套的运行在主线程中的onProgressUpdate方法。
```java
final ProgressBar progressBar = ...
progressBar.setMax(100);

// AsyncTask的三个类型参数分别是：输入、更新、返回
AsyncTask<Integer,Integer,Void> task = new AsyncTask<Integer,Integer,Void>(){
public void doInBackground(Integer ... params){
for(Integer progress:params){
publishProgress(progress);
Thread.sleep(1000);
}
}
public void onProgressUpdate(Integer...params){
int progress = params[0];
progressBar.setProgress(progress);
}
}
```
AsyncTask类是一个轻量级的多线程解决方案，主要应用于那些短暂且较少重复的任务。如果创建了大量的AsyncTask，或者长时间运行了AsyncTask，那么很可能是做出了错误的选择。此外，自Android3.2起，AsyncTask不再为每一个AsyncTask实例单独创建一个线程，而是使用一个Executor在单一的线程上运行所有的AsyncTask后台任务，意味着每个AsyncTask都需要排队逐个运行。


## SearchView
Android3.0之前的版本基于一个重叠在Activity上的对话框实现搜索界面及功能，具体过程略过。3.0以后的版本基于SearchView来实现。SearchView类属于操作视图，可内置在操作栏里。

定义搜索配置文件：res/xml/searchable.xml
搜索配置文件用来描述搜索对话框如何显示自身。
<?xml ...
<searchable xmlns:android=”http://...”
android:label=”@string/app_name”
android:hint=”@string/search_hint”
/>

将Activity定义为可搜索的Activity：
<manifest ...
<application ...
<activity
android:name=”.PhotoGalleryActivity”
android:launchMode=”singleTop” // 收到intent时，如果activity实例已经处在回退栈的顶端，则不创建新的activity，而直接路由新intent给现有activity。
android:label=”@string/title_activity_photo_gallery” >
<intent-filter>
<action android:name=”android.intent.action.MAIN” />
<category android:name=”android.intent.category.LAUNCHER” />
</intent-filter>
<intent-filter>  // 表明该activity可监听搜索intent
<action android:name=”android.intent.action.SEARCH” />
</intent-filter>
<meta-data  // 将搜索配置文件与目标activity关联起来
android:name=”android.app.searchable”
android:resource=”@xml/searchable”
/>
</activity>	
</application>
</manifest>

修改Activity，覆盖onNewIntent方法来处理搜索工作：
```java
public class PhotoGalleryActivity extends SingleFragmentActivity{
...
@Override
public void onNewIntent(Intent intent){
...
// 搜索、缓存、更新UI
}
}
```
在菜单中添加SearchView操作视图：res/menu/fragment_photo_gallery.xml
<menu xmlns:android=”http://...”>
		<item android:id=”@+id/menu_item_search”
android:title=”Search”
android:icon=”@android:drawable/ic_menu_search”
android:showAsAction=”ifRoom”
android:actionViewClass=”android.widget.SearchView”
/>
<item android:id=”@+id/menu_item_clear”
...
/>
</menu>
将菜单项的actionViewClass设为android.widget.SearchView，相当于告诉Android，不要在操作栏对此菜单项使用常规的视图部件，而是使用指定的视图类。SearchView将不会产生任何onOptionItemSelected(...)回调方法。

配置SearchView：
在发送搜索Intent之前，SearchView需要知道当前的搜索配置信息：通过SearchManager获取到一个SearchableInfo对象，并将它设置给SearchView。SearchableInfo包含了有关搜索的全部信息，包括应该接收intent的activity名称，以及所有searchable.xml中的配置信息。
```java
public class PhotoGalleryFragment extends Fragment{
...
	@Override
@TargetApi(11)
public void onCreateOptionMenu(Menu menu,MenuInflater inflater){
super.onCreateOptionsMenu(menu,inflater);
inflater.inflate(R.menu.fragment_photo_gallery,menu);
if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB){
MenuItem searchItem = menu.findItem(R.id.menu_item_search);
SearchView searchView = (SearchView)searchItem.getActionView();

SearchManager searchManager = 
(SearchManager)getActivity().getSystemService(Context.SEARCH_SERVICE);

// searchable activity的component name，由此系统可通过intent进行唤起
ComponentName name = getActivity().getComponentName();
SearchableInfo searchInfo = searchManager.getSearchableInfo(name);

searchView.setSearchableInfo(searchInfo);
}
}
}
```
SearchView相当于可搜索Activity在其他Activity中的一个入口，点击后展示可搜索Activity。菜单中的Item定义了SearchView的显示，而搜索配置文件searchable.xml定义了可搜索Activity的显示。

## 后台服务
服务就是Android应用的后台，即使前台关闭，activity长时间停止运行，后台服务依然可以持续不断地执行工作任务。
Service不是独立的线程，运行在主线程中。IntentService运行在单独的线程中。服务的Intent也称命令，IntentService接收到首个命令时，即完成启动，并触发一个后台线程，然后将命令放入队列。IntentService逐个执行命令队列里的命令，为每一条命令在后台线程上调用onHandleIntent方法。执行完队列中全部命令后，服务也随即停止并被销毁。

服务按启动方式可分为两种：
1.Started：即通过startService()方式启动的服务。将单独在后台运行，即使启动它的Component已经被销毁。一般用来执行一个单独的操作，操作执行结束后，Service即销毁，比如上传文件。
2.Bound即通过bindService()绑定到特定Component的方式来启动，提供了一个C/S方式的交互通道，使得所绑定的Component可以与Service进行交互：即可以发送request，接收result，甚至实现跨进程通信。多个Component可以绑定到同一个Service，一旦它们全部unbind，则该Service将自动销毁。
两种方式启动Service的生命周期：
  ![image](https://github.com/woojean/woojean.github.io/blob/master/images/android_5.png)

服务的类型由onStartCommand()方法的返回值决定，可能的返回值包括：Service.START_NOT_STICKY，START_REDELIVER_INTENT，START_STICK。

non-sticky服务：IntentService是一种non-sticky服务，在服务自己认为已完成任务时停止。no-sticky服务包括：START_NOT_STICKY,START_REDELIVER_INTENT，区别在于：如果系统要在服务完成任务之前关闭它，则服务的具体表现会有所不同，STRAT_NOT_STICK型服务会被关闭，而START_REDELIVER_INTENT型服务会在可用资源不再吃紧时，尝试再次启动服务。IntentService默认是START_NOT_STICK型服务，但是可以通过调用IntentService.setIntentRedelivery(true)方法来切换使用START_REDELIVER_INTENT。

sticky服务：会持续运行，直到外部组件调用Context.stopService(Intent)方法让它停止为止。适用于长时间运行，直到用户主动停止的服务。

延迟运行服务：
延迟运行服务的一种方式是调用Handler的sendMessageDelayed(...)或者postDelayed(...)方法。但如果用户离开当前应用，进程就会停止，Handler消息也会随之消亡，因此该解决方案不可靠。
应该使用AlarmManager结合PendingIntent来实现延迟运行服务：创建一个启动特定服务的Intent，然后用PendingIntent进行打包，并发送给AlarmManager。
即使进程停止了（如通过按后退键退出应用），AlarmManager依然会不断地发送intent，以反复启动服务。
一个PendingIntent只能登记一个定时器。

例：在后台查询新的搜索结果，一旦有了新的搜索结果，用户即可在状态栏收到通知信息。
创建一个IntentService：
```java
public class PollService extends IntentService{
private static final POLL_INTERVAL = 1000*15;

public PollService(){
}

@Override
protected void onHandleIntent(Intent intent){
// 检查后台网络是否可用：因为Android为用户提供了关闭后台应用网络连接的功能（节省流量、电量）
ConnectivityManager cm = 
(ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);

@SuppressWarning(“deprecation”)
// getBackgroundDataSetting()用于旧版本系统
// 要使用getActiveNetworkInfo()方法，需要添加：
<uses-permission android:name=”android.permission.ACCESS_NETWORK_STATE” />
boolean isNetworkAvailable = 
cm.getBackgroundDataSetting() && cm.getActiveNetworkInfo() != null;
if(!isNetworkAvailable) return;

// SharedPreferences中保存了最近一次的加载记录，可能有两种情况，一是没有使用查询条件，二是使用了查询条件。如果最近一次的加载记录（即SharedPreferences中当前存储的值）是使用了查询条件的，那么query就代表其查询关键字，而lastRequestId就是查询结果中的第一条记录的ID。
SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(this);
String query = prefs.getString(FlickrFetchr.PREF_SEARCH_QUERY,null);
String lastRequestId = prefs.getString(FlickrFetchr.PREF_LAST_RESULT_ID,null);

ArrayList<GalleryItem> items;
if(query != null ){
items = new FlickrFetchr().search(query); // 加载所有（其实就是第一页）
}else{
items = new FlickrFetchr().fetchItems(); // 根据关键字加载
}

if(items.size() == 0) return;  

String resultId = items.get(0).getId(); // 取查询结果中的第一条记录的ID，用于判断是否有新的查询结果

if(!resultId.equals(lastResultId)){
		// 得到一个新的查询结果，发出通知
Resources r = getResources();
PendingIntent pi = PendingIntent.getActivity(
this,
0,
new Intent(this,PhotoGalleryActivity.class),
0);
Notification notification = new NotificationCompat.Builder(this)
.setTicker(r.getString(R.string.new_pictures_title))
.setSmallIcon(android.R.drawable.ic_menu_report_image)
.setContentTitle(r.getString(R.string.new_pictures_title))
.setContentText(r.getString(R.string.new_pictures_text))
.setContentIntent(pi)
.setAutoCancel(true)
.build();
NotificationManager notificationManager = 
(NotificationManager)getSystemService(NOTIFICATION_SERVICE);
notificationManager.notify(0,notification); // 第一个整数参数是一个消息ID，整个应用中国值应该是唯一的。如果使用同一个ID发送两条消息，则第二条消息会替换掉第一条消息（进度条及其他动态视觉效果的实现方式）
}
else{
// 得到一个旧的查询结果
}

prefs.edit().putString(FlickrFetchr.PREF_LAST_RESULT_ID,resultId).commit();
}

// 实现延迟运行服务的方法
public static void setServiceAlarm(Context context,boolean isOn){
Intent i = new Intent(context,PollService.class);
PendingIntent pi = PendingIntent.getService(context,0,i,0); // 如果使用同一个intent请求PendingIntent两次，得到的PendingIntent仍会是同一个，可以借此测试一个PendingIntent是否已存在，或撤销已发出的PendingIntent

AlarmManager alarmManager = 
(AlarmManager)context.getSystemService(Context.ALARM_SERVICE);
if(isOn){
alarmManager.setRepeating(  // 设置定时器
AlarmManager.RTC, // 硬件闹钟，不唤醒设备休眠，即当设备休眠时不发射闹钟
System.currentTimeMillis(),
POLL_INTERVAL,
pi
);
}else{ // 取消定时器
alarmManager.cancel(pi);
pi.cancel();
}
}

// 如果使用同一个intent请求PendingIntent两次，得到的PendingIntent仍会是同一个
public static boolean isServiceAlarmOn(Context context){
Intent i = new Intent(context,PollService.class);
// 一个PendingIntent只能登记一个定时器
PendingIntent pi = PendingIntent.getService(
context,
0,
i,
PendingIntent.FLAG_NO_CREATE)
);
return pi!=null;
}
}
```

在AndroidManifest.xml中声明Service：
	<manifest
<application ... 
<activity ... 
</activity
<service android:name=”.PollService” />

根据服务的启动状态更新选项菜单的状态：
在3.0以前的版本上，除了菜单的首次创建外，每次菜单需要配置都会调用onPrepareOptionMenu()方法，因此可以用来更新选项菜单的内容。在3.0以后的版本中，操作栏无法自动更新自己，需要通过调用Activity.invalidateOptionMenu()方法来回调onPrepareOptionMenu()方法，并刷新菜单项。
```java
public class PhotoGalleryFragment extends Fragment{
...
@Override
public void onPrepareOptionMenu(Menu menu){
super.onPrepareOptionsMenu(menu);
MenuItem toggleItem = menu.findItem(R.id.menu_item_toggle_polling);
if(PollService.isSeviceAlarmOn(getActivity()){
toggleItem.setTitle(R.string.stop_polling);
}
else{
toggleItem.setTitle(R.string.start_polling);
}
}

@Override
@TargetApi(11)
public boolean onOptionsItemSelected(MenuItem item){
switch(item.getItemId()){
...
case R.id.menu_item_toggle_polling:
// 切换服务启动状态
boolean shouldStartAlarm = !PollService.isServiceAlarmOn(getActivity());
PollService.setServiceAlarm(getActivity(),shouldStartAlarm);

// 刷新菜单
if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB)
getActivity().invalidateOptionsMenu(); // 回调onPrepareOptionMenu()方法
return true;
default:
return super.onOptionItemSelected(item);
}
}
}
```
## Broadcast Intent
Broadcast Intent可以同时被多个组件接收，Broadcast Receiver负责接收各种Broadcast Intent。
 ![image](https://github.com/woojean/woojean.github.io/blob/master/images/android_6.png)

例：实现随设备重启而重启的定时器
设备重启后，那些持续运行的应用通常也需要重启，通过监听具有BOOT_COMPLETE操作的Broadcast Intent可得知设备是否已完成启动。
BroadcastReceiver与Service、Activity一样，属于可以接收Intent的组件，需要在系统中登记。

修改之前的Service的代码，在设置Service启停的方法中往Preferences中加入一个定时器当前启停状态的标识：
```java
public class PollService extends IntentService{
public static void setServiceAlarm(Context context,boolean isOn){
...
PreferenceManager.getDefaultSharedPreferences(context)
.edit()
.putBoolean(PollService.PREF_IS_ALARM_ON，isOn) // 保存定时器的启停状态
.commit();
}
}
```
创建一个broadcast receiver：
```java
public class StartupReceiver extends BroadcastReceiver{

@Override
public void onReceive(Context context,Intent intent){
SharedPreferences prefs =  PreferencesManager.getDefaultSharedPreferences(context);
boolean isOn = prefs.getBoolean(PollService.PREF_IS_ALARM_ON,false);
PollService.setServiceAlarm(context,isOn);
}
}
```
在Manifest文件中登记broadcast receiver：
在配置文件中完成声明后，即使应用当前并未运行，只要有匹配的broadcast intent发来，broadcast receiver就会接收，并执行onReceive方法，broadcast receiver随即被销毁。因为broadcast receiver的存在非常短暂，因此其作用有限，例如无法使用任何异步API或登记任何监听器，因为onReceive(...)方法刚运行完，receiver就不存在了。此外，onReceive(...)方法运行在主线程上，因此不能在该方法内做一些耗时的任务，如网络连接，数据读写存储等。broadcast receiver适合处理一些轻型的任务，比如这里的重置定时器。
<manifest ...>
...
<uses-permission android:name=”android.permission.RECEIVE_BOOT_COMPLETED” />
...
<application>
...
<receiver android:name=”.StartupReceiver”>
<intent-filter>
<action android:name=”android.intent.action.BOOT_COMPLETED” />
</intent-filter>
</receiver>
</application>
</manifest>

例：动态的Broadcast Receiver
目前的通知消息存在一个问题：应用的通知消息虽然工作良好，但是在打开应用后依然会收到通知消息（即，在使用某个应用的过程中不应该再收到该应用的新通知）。

解决思路：
有两个receiver，一个静态注册，用来实际发送消息，其优先级非常低。另有一个动态注册，即在特定Fragment的onResume()中注册，在onPause()中注销。
因为应用监听了系统重启事件，并开启定时器，所以只要打开设备，检查是否有图片更新的后台服务就会被定时器触发去运行。当发现有新的图片时，会以Activity.RESULT_OK为resultCode来执行sendOrderedBroadcast(...)。而在实际执行发送消息的receiver中会通过getResultCode()获取resultCode并进行判断，如果是Activity.RESULT_OK，则执行消息发送操作，否则直接return。
而动态注册的receiver也接收同样的action，可以处理同样的broadcast intent，且优先级更高，只要当前应用打开，则会创建加载指定Fragment，因此动态receiver得以在Fragment调用onResume()时进行注册，之后只要应用保持活动状态，该动态注册的receiver就会一直保持监听状态。当后台线程再次发出有序broadcast时，会被该动态注册的，具有更高优先级的receiver接收并处理：把resultCode改成Activity.CANCEL。之后，当实际负责发送消息的receiver处理broadcast intent时，便不会再发出消息。


发送broadcast intent：
```java
public class PollService extends IntentService{
...
// 定义一个操作常量
public static final String ACTION_SHOW_NOTIFICATION = “com.xxx.xx...”;
...
@Override
protected void onHandleIntent(Intent intent){
if(!resultId.equals(lastResultId)){
		// 得到一个新的查询结果，发出通知
...
notificationManager.notify(0,notification); 

// 广播我们定义的操作
sendBroadcast(new Intent(ACTION_SHOW_NOTIFICATION));
}
...
}
}
```
接收broadcast intent：
这里静态的receiver行不通，因为需要在PhotoGalleryFragment存在的时候才接收intent，静态的receiver很难做到在不断接收intent的同时还有确定PhotoGalleryFragment的存在状态。

新建一个用于隐藏前台通知的通用fragment：
```java
public abstract class VisibleFragment extends Fragment{
private BroadcastReceiver mOnShowNotification = new BroadcastReceiver(){
@Override
public void onReceive(Context context,Intent intent){
Toast.makeText(
getActivity(),
”获取到一个新的广播信息:”+intent.getAction(),
Toast.LENGTH_LONG).show();
}
};
// 在onResume()中注册receiver
@Override
public void onResume(){
super.onResume();
// 用一个动作名称来顶一个IntentFilter
IntentFilter filter = new IntentFilter(PollService.ACTION_SHOW_NOTIFICATION);
// 用一个BroadcastReceiver + 一个IntentFilter来注册一个register
getActivity().registerReceiver(mOnShowNotification,filter);
}

// 在onPause()中注销receiver
@Override
public void onPause(){
super.onPause();
getActivity().unregisterReceiver(mOnShowNotification);
}
} 
```
因为设备发生旋转时，Fragment的onCreate()和onDestroy()方法中返回的getActivity()不同，因此，如果想在onCreate()和onDestroy()中实现登记或者注销登记，应该使用getActivity().getApplicationContext()方法。

修改PhotoGalleryFragment类，使其继承于VisibleFragment：
public class PhotoGalleryFragment extends VisibleFragment{
...
}

使用私有权限：
目前系统中的任何应用都可以触发上面定义的动态receiver。如果receiver声明在manifest配置文件中，且仅限应用内部使用，则可在receiver标签上添加一个android:exported=”false”属性，这样系统中的其他应用就再也无法接触到该receiver。此外还可以通过创建自己的权限来进行限制。
<manifest
...
// 定义私有权限
<permission 
android:name=”com.xxx...”  // 与发送broadcast intent时定义的操作常量相同
android:protectionLevel=”signature” />
...
// 使用私有权限
<uses-permission
android:name=”com.xxx...” />
...
</manifest>

PollService中发送广播时的代码修改为：
public static final String PERM_PRIVATE = “com.xxx...”;
...
	sendBroadCast(new Intent(ACTION_SHOW_NOTIFICATION),PERM_PRIVATE); // 发送广播时带上权限，任何应用必须使用相同的权限才能接收该Intent。

在VisibleFragment中注册监听器时也指定权限：
getActivity().registerReceiver(mOnShowNotification,filter,PollService.PERM_PRIVATE,null);

权限本身只是一行简单的字符串，它需要出现在三个地方：
1.发送广播的时候
2.定义私有权限的时候
3.使用私有权限的时候

自定义的权限必须指定android:protectionLevel属性值，Android根据该值来确定自定义权限的使用方式，如上面android:protectionLevel=”signature”，则如果其他应用想要使用我们的自定义权限，必须使用和当前应用相同的key做签名认证。protectionLevel的可选值包括：
normal:应用安装前，用户会看到相应的安全级别，但无需主动授权，主要用来告诉用户可能带来的影响。RECEIVE_BOOT_COMPLETED、手机震动等使用该安全级别。
dangerous：normal安全级别控制以外的任何危险操作，如访问个人隐私、通过网络接口收发数据、使用可监视用户的硬件功能等。需要用户的明确授权。网络使用权限、相机使用、联系人信息使用等都属于该级别。
signature：如果应用签署了与声明应用一致的权限证书，则该权限由系统授予，否则系统作相应的拒绝。授予权限时，系统不会通知用户，通常适用于应用内部。
signatureOrSystem：类似于signature，但该授权级别针对Android系统镜像中的所有包授权，用于系统镜像内应用间的通信，用户通常无需关心。

使用ordered broadcast实现双向通信：
理论上一个broadcast intent可被多个receiver同时接收并处理，不能指望它们按照某种顺序依次运行，也无法知道它们什么时候全部结束运行。有序broadcast允许多个broadcast receiver依序处理broadcast intent，此外通过传入一个result receiver，有序broadcast还可以实现让broadcast的发送者接收broadcast接收者的返回值。
修改broadcast receiver的onReceive()方法，将取消通知的信息发送给broadcast的发送者：
```java
public abstract class VisibleFragment extends Fragment{
private BroadcastReceiver mOnShowNotification = new BroadcastReceiver(){
@Override
public void onReceive(Context context,Intent intent){
setResultCode(Activity.RESULT_CANCELED); // 还有其他的可选方法，如setResultData(String)、setResultExtras(Bundle)
}
};
...
```
要使该取消操作有效，broadcast必须有序。在PollService中新增一个可发送有序broadcast的新方法：
```java
public class PollService extends IntentService{
...
void showBackgroundNotification(int requestCode, Notification notification){
Intent i = new Intent(ACTION_SHOW_NOTIFICATION);
i.putExtra(“REQUEST_CODE”,requestCode);
i.putExtra(“NOTIFICATION”，notification);
sendOrderedBroadcast(i,PERM_PRIVATE,null,null,Activity.RESULT_OK,null,null);
}
...
sendOrderedBroadcast方法原型：
public abstract void sendOrderedBroadcast (
Intent intent, 
String receiverPermission, 
BroadcastReceiver resultReceiver,  
Handler scheduler, 
int initialCode, 
String initialData, 
Bundle initialExtras)
```
其中resultReceiver只在所有有序broacast intent的接收者运行结束后才开始运行，scheduler为一个支持resultReceiver运行的Handler。

新建一个处理有序broadcast的receiver：
```java
		public class NotificationReceiver extends BroadcastReceiver{

@Override
public void onReceive(Context c,Intent i){
if(getResultCode()!= Activity.RESULT_OK)
return;
int requestCode = i.getIntExtra(“REQUEST_CODE”,0);
Notification notification = (Notification)i.getParcelableExtra(“NOTIFICATION”);
NotificationManager notificationManager = 
(NotificationManager)c.getSystemService(Context.NOTIFICATION_SERVICE);
notificationManager.notify(requestCode,notification);
}
}
```
登记新建的receiver：
因为NotificationReceiver接收其他receiver返回的结果码，并负责发送消息，它的运行应该总是在最后，这需要将其优先级设置为最低，即设置其优先级为-999（-1000及以下值属于系统保留值）。
<manifest
<application
...
<receiver 
android:name=”.NotificationReceiver”
android:exported=”false”>
<intent-filter android:priority=”-999”>
<action android:name=”com.xxx...SHOW_NOTIFICATION” />
</intent-filter>
</receiver>
</application>
</manifest>

修改PollService中获取到新的图片时发送通知的方式：
```java
	public class PollService extends IntentService{
...
@Override
protected void onHandleIntent(Intent intent){
if(!resultId.equals(lastResultId)){
		// 得到一个新的查询结果，发出通知
...
Notification notification = new NotificationCompat.Builder(this)...build();
showBackgroundNotification(0,notification);
}
```
## 网页浏览
使用浏览器应用：
```java
Uri photoPageUri = Uri.parse(item.getPhotoPageUrl());
Intent i = new Intent(Intent.ACTION_VIEW,photoPageUri);  // 隐式Intent
startActivity(i);
```
使用WebView:
定义一个包含WebView的Fragment：
```java
public class PhotoPageFragment extends VisibleFragment{
private String mUrl;
private WebView mWebView;

@Override
public void onCreate(Bundle savedInstanceState){
		super.onCreate(savedInstanceState);
setRetainInstance(true);

mUrl = getActivity().getIntent().getData().toString();
}

@SuppressLint(“SetJavaScriptEnabled”)
@Override
public View onCreateView(LayoutInflater inflater,ViewGroup parent,Bundle savedInstanceState){
View v = inflater.inflate(R.layout.fragment_photo_page,parent,false);
mWebView = (WebView)v.findViewById(R.id.webView);

mWebView.getSettings().setJavaScriptEnabled(true);  // 启用Javascript

// WebViewClient是响应渲染事件的接口，WebChromeClient是响应那些改变浏览器中装饰元素的事件接口
mWebView.setWebViewClient(new WebViewClient(){
// shouldOverrideUrlLoading(...)方法用于处理当前WebView中存在链接的情况
如果WebView根本没有设置WebViewClient，则在WebView中点击url时，会弹出可选应用的列表。如果设置了WebViewClient，则当该方法返回true时，表示点击链接后将离开当前WebView，由应用来处理接下来的步骤。如果返回false，则表示点击链接后仍然在该WebView中打开新链接。
public boolean shouldOverrideUrlLoading(WebView view,String url){
return false;
}
}

final ProgressBar progressBar = (ProgressBar)v.findViewById(R.id.progressBar);
progressBar.setMax(100);
final TextView titleTextView = (TextView)v.findViewById(R.id.titleTextView);
mWebView.setWebChromeClient(new WebChromeClient(){
public void onProgressChanged(WebView webView,int progress){
if(progress == 100){
progressBar.setVisibility(view.INVISIBLE);
}
else{
progressBar.setVisibility(View.VISIBLE);
pregressBar.setProgress(progress);
}
}

public void onReceivedTitle(WebView webView,String title){
titleTextView.setText(title);
}
}

mWebView.loadUrl(mUrl);
return v;
}
}
```
创建托管该Fragement的Activity：
```java
public class PhotoPageActivity extends SingleFragmentActivity{
@Override
public Fragment createFragment(){
return new PhotoPageFragment();
}
}
```
之后，在其他地方启动该Activity：
```java
Intent i = new Intent(getActivity(),PhotoPageActivity.class);
i.setData(photoPageUri);
startActivity(i);
```
解决旋转问题：
当设备旋转时，WebView必须重新加载网页，因为WebView包含了太多的数据，以至于无法在onSaveInstanceState(...)方法内保存所有数据。因此每次设备旋转，它都必须重头开始加载网页数据。可以通过设置android:configChanges属性来通知Activity自己处理设备配置调整，即无需销毁重建activity，可直接调整自己的视图已适应新的屏幕尺寸。
<activity 
android:name=”.PhotoPageActivity”
android:configChanges=”keyboardHidden|orientation|screenSize” />
...
默认情况下，当设备配置发生变更时，activity会销毁并重建。设置了configChanges属性后，当设备配置发生变更时，不会销毁activity，而是会调用其onConfigurationChanged()方法。

## 定制视图与触摸事件
定义一个表示矩形框的类：
```java
public class Box{
private PointF mOrigin;
private PointF mCurrent;
public Box(PointF origin){
mOrigin = mCurrent = origin;
}
public void setCurrent(PointF current){
mCurrent = current;
}
public PointF getCurrent(){
return mCurrent;
}
public PointF getOrigin(){
return mOrigin;
}
}
```
创建自定义视图类：
处理触摸事件有两种方式，一是使用View.OnTouchListener接口，实现onTouch(...)方法，然后在视图上调用setOnTouchListener(View.OnTouchListener)方法。另一种方式是直接覆盖View的onTouchEvent(...)方法。前一种方式的优先级更高，如果同时使用两种方式，则只有在onTouch()方法中返回false时，onTouchEvent(...)方法才会被接着调用。
```java
public class BoxDrawingView extends View{
private Box mCurrentBox;
private ArrayList<Box> mBoxes = new ArrayList<Box>();
private Paint mBoxPaint;
private Paint mBackgroundPaint;

public BoxDrawingView(Context context){ // 从代码实例化
this(context,null);
}

public BoxDrawingView(Context context,AttributeSet attrs){ // 从布局文件实例化
super(context,attrs);
mBoxPaint = new Paint();
mBoxPaint.setColor(0x22ff0000);
mBackgroundPaint = new Paint();
mBackgroundPaint.setColor(0xfff8efe0);
}

public boolean onTouchEvent(MotionEvent event){
PointF curr = new PointF(event.getX(),event.getY());
switch(event.getAction()){
case MotionEvent.ACTION_DOWN:
		// 按下后，初始化一个box，起点和终点一样，之后起点不变，终点随MOVE不断变化，直至UP
mCurrentBox = new Box(curr);
mBoxes.add(mCurrentBox);
break;
case MotionEvent.ACTION_MOVE:
if(mCurrentBox != null){
mCurrentBox.setCurrent(curr);
invalidate(); // 强制视图重绘
}
break;
case MotionEvent.ACTION_UP:
mCurrentBox = null;
break;
case MotionEvent.ACTION_CANCEL:
mCurrentBox = null; 
break;
}
return true;
}

// 覆盖视图的onDraw(...)方法
@Override
protected void onDraw(Canvas canvas){
canvas.drawPaint(mBackgroundPaint);
for(Box box:mBoxes){
float left = Math.min(box.getOrigin().x,box.getCurrent().x);
float right = Math.max(box.getOrigin().x,box.getCurrent().x);
float top= Math.min(box.getOrigin().y,box.getCurrent().y);
float bottom= Math.min(box.getOrigin().y,box.getCurrent().y);

canvas.drawRect(left,top,right,bottom,mBoxPaint);
}
}
}
```
在Fragment布局文件中引用自定义视图类：
fragment_drag_and_draw.xml
<com.xxx...BoxDrawingView // 必须使用全路径类名，否则布局inflater会在android.view和android.widget包中寻找目标。对于android.view和android.widget包以外的自定义视图类，必须指定其全路径类名。
xmlns:android=”http://xxx...”
android:layout_width=”match_parent”
android:layout_height=”match_parent”
/>

创建Fragment：
```java
public class DragAndDrawFragment extends Fragment{
@Override
public View onCreateView(LayoutInflater inflater,ViewGroup parent,Bundle savedInstanceState){
View v =inflater.inflate(R.layout.fragment_drag_and_draw,parent,false);
return v;
}
}
```
创建托管Fragment的Activity：
```java
public class DragAndDrawActivity extends SingleFragmentActivity{
@Override
public Fragment createFragment(){
return new DragAndDrawFragement();
}
}
```

## 跟踪设备的地理位置
Android中的地理位置数据是由LocationManager系统服务提供的，该系统服务向所有需要地理位置数据的应用提供数据更新，通常采用两种方式：
1.使用LocationListener接口，实现onLocationChanged(...)方法。
2.使用PendingIntent获取地理位置数据更新，然后发布广播。
第1种方法在实现LocationListener接口的组件不可用时就无法进行地理位置数据更新。而第2种方法即使应用组件，甚至整个应用进程都销毁了，LocationManager仍然会一致发送Intent，直到要求它停止并按需启动新组件响应它们。

创建一个与LocationManager通信的单例类：
```java
public class RunManager{
public static final String ACTION_LOCATION = “xxx....”;
private static RunManager sRunManager;
private Context mAppContext;
private LocationManager mLocationManager;

private RunManager(Context appContext){
mAppContext = appContext;
mLocationManager = 
(LocationManager)mAppContext.getSystemService(Context.LOCATION_SERVICE);
}

// 单例方法，返回实例
public static RunManager get(Context c){  
if(sRunManager == null){
sRunManager = new RunManager(c.getApplicationContext());
}
return sRunManager;
}

// 返回一个新的或者已存在的PendingIntent
private PendingIntent getLocationPendingIntent(boolean shouldCreate){
Intent broadcast = new Intent(ACTION_LOCATION);
int flag = shouldCreate ? 0 :PendingIntent.FLAG_NO_CREATE; // FLAG_NO_CREATE;如果当前系统中不存在相同的所描述的PendingIntent对象，系统将不会创建该PendingIntent对象而是直接返回 null
return PendingIntent.getBroadcast(mAppContext,0,broadcast,flags);
}

// 启动位置更新：构造一个PendingIntent，发送给LocationManager
public void startLocationUpdates(){
String provider = LocationManager.GPS_PROVIDER; // 还有WIFI、手机基站等定位方式

// 获取最近一次的地理位置,然后模拟LocationManager的方式发送广播
Location lastKnown = mLocationManager.getLastKnownLocation(provider);
if(lastKnown != null){
lastKnown.setTime(System.currentTimeMillis());
Intent broadcast = new Intent(ACTION_LOCATION);
broadcast.putExtra(LocationManager.KEY_LOCATION_CHANGED，location);
mAppContext.sendBroadcast(broadcast);
}

PendingIntent pi = getLocationPendingIntent(true);
mLocationManager.requestLocationUpdates(provider,0,0,pi); // 一旦有位置更新，就把这个PendingIntent发出去
}

// 停止位置更新
public void stopLocationUpdates(){
PendingIntent pi = getLocationPendingIntent(false);
if( pi != null){
mLocationManager.removeUpdates(pi);
pi.cancel();
}
}

public boolean isTrackingRun(){
return getLocationPendingIntent(false) != null;
}
}
```
创建用于接收位置信息更新的receiver：
```java
public class LocationReceiver extends BroadcastReceiver{
@Override 
public void onReceive(Context context,Intent intent){
		// 启动LocationManager的requestLocationUpdates(...)方法时，传入了一个PendingIntent
LocationManager在发生位置更新事件时想必是先往该PendingIntent中装入了一个索引为KEY_LOCATION_CHANGED的Location实例后，再发送的
Location loc = 
(Location)intent.getParcelableExtra(LocationManager.KEY_LOCATION_CHANGED);

if(loc != null){
onLocationReceived(context,loc);
return;
}
if(intent.hasExtra(LocationManager.KEY_PROVIDER_ENABLED)){
boolean enabled = intent.getBooleanExtra(LocationManager.KEY_PROVIDER_ENABLES,false);
onProviderEnabledChanged(enabled);
}
}

protected void onLocationReceived(Context context,Location loc){
// 地理位置有更新 
}

protected void onProviderEnabledChanged(boolean enabled){
// 定位服务启停状态有更新
}
}
```
添加地理位置使用权限和receiver：
<manifest
...
<uses-permission android:name=”android.permission.ACCESS_FINE_LOCATION” />
<uses-feature 
android:required=”true”
android:name=”android.hardware.location.gps”
/>
...
<application
...
<receiver
android:name=”.LocationReceiver”
android:exported=”fasle” >
<intent-filter>
<action android:name=”com....ACTION_LOCATION” />
</intent-filter>
</receiver>

实现一个简单的保存开始日期的Run类：
```java
public class Run{
private Date mStartDate;
public Run(){
mStartDate = new Date();
}
public Date getStartDate(){...
public void setStartDate(Date startDate...
public int getDurationSeconds(long endMillis..
}
```
使用地理位置更新UI：
```java
public class RunFragment extends Fragment{
private Button mStartButton,mStopButton;
private TextView mStartedTextView,mLatitudeTextView,mLongitudeTextView,mAltitudeTextView,mDurationTextView;

private RunManger mRunManager;
private Run mRun;
private Location mLastLocation;

// 实现一个继承自LocationReceiver的匿名内部类，重写相关方法
private BroadcastReceiver mLocationReceiver = new LocationReceiver(){
@Override
protected void onLocationReceived(Context context,Location loc){
mLastLocation = loc;
if(isVisible())
updateUI();
}

@Override
protected void onProviderEnabledChanged(boolean enabled){
// toast一下状态变化
}
};


@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
setRetainInstance(true);
mRunManager = RunManager.get(getActivity());
}

@Override
public void onStart(){
super.onStart();
getActivity().registerReceiver(
mLocationReceiver,
new IntentFilter(RunManager.ACTION_LOCATION)
);
}

@Override
public void onStop(){
getActivity().unregisterReceiver(mLocationReceiver);
super.onStop();
}

@Override
public View onCreateView(LayoutInflater inflater,ViewGroup container,Bundle savedInstanceState){
View = inflater.inflate(...
// 初始化控件，略

mStartButton.setOnClickListener(new View.OnClickListener(){
@Override
public void onClick(View v){
mRunManager.startLocationUpdates();
mRun = new Run();
updateUI();
}
});

mStopButton.setOnClickListener(new View.OnClickListener(){
@Override
public void onClick(View v){
mRunManager.stopLocationUpdates();
updateUI();
}
});

uodateUI();
return v;
}

// 使用从LocationManager中返回的Location对象中的数据更新UI
private void updateUI(){
boolean started = mRunManager.isTrackingRun();

if(mRun != null)
mStartedTextView.setText(mRun.getStartDate().toString());
int durationSeconds = 0;
if(mRun != null && mLastLocation != null){
durationSeconds  = mRun.getDurationSeconds(mLastLocation.getTime());
mLatitudeTextView.setText(Double.toString(mLastLocation.getLatitude());
mLongitudeTextView.setText(Double.toString(mLastLocation.getLongitude());
mAltitudeTextView.setText(Double.toString(mLastLocation.getAltitude());
}
...
mStartButton.setEnabled(!started);
mStopButton.setEnabled(started);

}

}
```

## 使用SQLite本地数据库
 ![image](https://github.com/woojean/woojean.github.io/blob/master/images/android_7.png)

新建一个数据库操作辅助类：
```java
public class RunDatabaseHelper extends SQLiteOpenHelper{
private static final String DB_NAME =”runs.sqlite”; // 数据库名
private static final int VERSION = 1;

private static final String TABLE_RUN = “run”; // 表名
private static final String COLUMN_RUN_START_DATE = “start_date”; // 列名

private static final String TABLE_LOCATION = “location”; // 表名
private static final String COLUMN_LOCATION_LATITUDE = “latitude”; // 列名
...其他列名略


public RunDatabaseHelper(Context context){
super(context,DB_NAME,null,VERSION);  // 这里使用数据库名调用超类构造方法创建数据库
}

@Override
public void onCreate(SQLiteDatabase db){
db.execSQL(“create table run(“ // 创建run表
+”_id integer primary key autoincrement,start_date integer)”);
db.execSQL(“create table location(“ // 创建location表
+”timestamp integer,latitude real,longtitude real,altitude real,” // real是浮点数
+”provider varchar(100),run_id integer references run(_id))”);
}

@Override
public void onUpgrade(SQLiteDatabase db,int oldVersion, int newVersion){
// 数据库迁移代码，实现不同版本间的数据库结构升级或转换
}

// 往run表中插入数据
public long insertRun(Run run){
ContentValues cv = new ContentValues();
cv.put(COLUMN_RUN_START_DATE,run.getStartDate().getTime());
return getWritableDatabase().insert(TABLE_RUN,null,cv); // 往TABLE_RUN表中插入cv数据
}

// 往location表中插入数据
public long insertLocation(long runId,Location location){
ContentValues cv = new ContentValues();
cv.put(COLUMN_LOCATION_LATITUDE,location.getLatitude());
...
return getWritableDatabase().insert(TABLE_LOCATION,null,cv); // 返回新插入记录的ID
}

}
```
为Run类添加ID属性：
```java
public class Run{
private long mId;
private Date mStartDate;
public Run(){
mStartDate = new Date();
}
public Date getStartDate(){...
public void setStartDate(Date startDate...
public int getDurationSeconds(long endMillis..
}
```

修改RunManager类以使用数据库：
```java
public class RunManager{
private static final String PREFS_FILE = “runs”;
private static final String PREF_CURRENT_RUN_ID = “RunManager.currentRunId”;
private RunDatabaseHelper mHelper;
private SharedPreferences mPrefs;
private long mCurrentRunId;

public static final String ACTION_LOCATION = “xxx....”;
private static RunManager sRunManager;
private Context mAppContext;
private LocationManager mLocationManager;

private RunManager(Context appContext){
mAppContext = appContext;
mLocationManager = 
(LocationManager)mAppContext.getSystemService(Context.LOCATION_SERVICE);

mHelper = new RunDatabaseHelper(mAppContext);
mPrefs = mAppContext.getSharedPreferences(PREFS_FILE,Context.MODE_PRIVATE);
mCurrentRunId = mPrefs.getLong(PREF_CURRENT_RUN_ID,-1）;
}

...

// 插入run记录
private Run insertRun(){
Run run = new Run();
run.setId(mHelper.insertRun(run));
return run;
}

// 插入Location记录
public void insertLocation(Location loc){
if(mCurrentRunId != 1){
mHelper.insertLocation(mCurrentRunId,loc);	
}
else{
...
}
}

public Run startNewRun(){
Run run = insertRun();
startTrackingRun(run);
return run;
}

public void startTrackingRun(Run run){
// 将Run实例传入的ID分别存储在实例变量和shared preferences中，这样即使在应用完全停止的情况下仍可重新取回ID
mCurrentRunId = run.getId();
mPrefs.edit().putLong(PREF_CURRENT_RUN_ID,mCurrentRunId).commit();
startLocationUpdates();
}

public void stopRun(){
stopLocationUpdates();
mCurrentRunId = -1;
mPrefs.edit().remove(PREF_CURRENT_RUN_ID).commit();
}
}
```
更新启停按钮代码：
```java
public class RunFragment extends Fragment{
...
@Override
public View onCreateView(LayoutInflater inflater,ViewGroup container,Bundle savedInstanceState){
...
mStartButton.setOnClickListener(new View.OnClickListener(){
@Override
public void onClick(View v){
mRunManager.startLocationUpdates();
mRun = new Run();
mRun = mRunManager.startNewRun();
updateUI();
}
});

mStopButton.setOnClickListener(new View.OnClickListener(){
@Override
public void onClick(View v){
mRunManager.stopLocationUpdates();
mRunManager.stopRun();
updateUI();
}
});
```
新建一个独立的Broadcast-Receiver来调用RunManager的insertLocation(Location)方法：
```java
public class TrackingLocationReceiver extends LocationReceiver{
@Override
protected void onLocationReceived(Context c,Location loc){
RunManager.get(c).insertLocation(loc);
}
}
```
将manifest中的<receiver android:name=”.LocationReceiver”改成<receiver android:name=”.TrackingLocationReceiver”，因为现在已经不直接使用LocationReceiver了。


使用CursorAdapter显示旅程列表：
首先为RunDatabaseHelper添加查询数据库的方法：
```java
public class RunDatabaseHelper extends SQLiteOpenHelper{
...
private static final String COLUMN_RUN_ID = “_id”;
...
// 继承CursorWrapper，实现一个封装Cursor的类。
因为Cursor将结果集看成是一系列的数据行和数据列，但仅支持String以及原始数据类型的值，而这里我们想要它能够返回Run对象的实例。
CursorWrapper类将封装Cursor，并转发所有的方法调用给它
public static class RunCursor extends CursorWrapper{
public RunCursor(Cursor c){  // 构造方法中传入一个Cursor
super(c);
}

public Run getRun(){
if(isBeforeFirst() || isAfterLast())
return null;
Run run = new Run();
long runId = getLong(getColumnIndex(COLUMN_RUN_ID));
run.setId(runId);
long startDate = getLong(getColumnIndex(COLUMN_RUN_START_DATE));
run.setStartDate(new Date(startDate));
return run;
}
}

public RunCursor queryRuns(){
Cursor wrapped = 
getReadableDatabase().
query(TABLE_RUN,null,null,null,null,null,COLUMN_RUN_START_DATE+”asc”);
return new RunCursor(wrapped);
}

在RunManager类中添加查询Run的方法：
public class RunManager{
...
public RunCursor queryRuns(){
return mHelper.queryRuns();
}
...
```

创建用于展示Run列表的Fragment：
```java
public class RunListFragment extends ListFragment{
private RunCursor mCursor;

@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
mCursor = RunManager.get(getActivity()).queryRuns();

// 这里在主线程中查询数据，设置adapter，显然是不好的实现
RunCursorAdapter adapter = new RunCursorAdapter(getActivity(),mCursor);
setListAdapter(adapter);
}

@Override 
public void onDestroy(){
mCursor.close();
super.onDestroy();
}

// 实现一个CursorAdapter类
private static class RunCursorAdapter extends CursorAdapter{
private RunCursor mRunCursor;
public RunCursorAdapter(Context context,RunCursor cursor){
super(context,cursor,0);
}

@Override
public View newView(Context context,Cursor cursor,ViewGroup parent){
LayoutInflater inflater = 
(LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
return inflater.inflate(android.R.layout.simple_list_item_1,parent,false);
}

// 当需要配置视图显示cursor中的一行数据时，CursorAdapter将调用bingView(...)方法，该方法中的View参数就是newView()方法中返回的View。
@Override
public void bindView(View view,Context context,Cursor cursor){
Run run = mRunCursor.getRun();  // getRun()方法为CursorWrapper封装Cursor后新增的方法
TextView startDateTextView = (TextView)view;
String cellText = context.getString(R.string.cell_text,run.getStartDate());
startDateTextView.setText(cellText);
}
}
}
```
## 使用Loader加载异步数据
Loader用来在activity或者fragment中异步加载数据。它有如下一些特性：
1.对每个activity或fragment都是可用的；
2.提供数据的异步加载功能；
3.控制数据源，并且当数据源内容发生变化时传送新的结果；
4.当设备配置发生改变时，能够自动重新连接至之前的状态；

LoadManager是一个用来管理一个或多个Loader实例的虚拟类。每一个activity或fragment都只包含一个LoadManager。LoaderManager在Activity和Fragment的生命周期中管理Loaders，并且在配置变化时保持已载入的数据

LoaderManager.LoaderCallbacks：LoadManager的回调接口，被客户端用来与LoadManager进行互动。该接口包含如下方法：
// 使用给定ID创建并返回一个Loader
abstract Loader<D>	onCreateLoader(int id, Bundle args)
 
// 当Loader完成数据加载时调用
abstract void	onLoadFinished(Loader<D> loader, D data)
 
// 当Loader重置时调用
abstract void	onLoaderReset(Loader<D> loader)

Loader的继承关系：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/android_8.png)
 

创建一个类型参数为Cursor的AsyncTaskLoader的子类：
```java
public abstract class SQLiteCursorLoader extends AsyncTaskLoader<Cursor>{
private Cursor mCursor;
public SQLiteCursorLoader(Context context){
supere(context);
}
protected abstract Cursor loadCursor();

@Override
public Cursor loadInBackground(){
Cursor cursor = loadCursor();
if(cursor != null){
cursor.getCount(); // 保证数据在发送给主线程之前已加载到内存中
}
return cursor;
}

@Override
public void deliverResult(Cursor data){
Cursor oldCursor = mCursor;
mCursor = data;
if(isStarted()){
super.deliverResult(data);
}
// 在关闭旧的cursor之前，必须确认新旧cursor并不相同
if(oldCursor != null && oldCursor != data && !oldCursor.isClosed()){
oldCursor.close();
}
}

@Override
protected void onStartLoading(){
if(mCursor != null ){
deliverResult(mCursor);
}
if(takeContentChanged() || mCursor == null ){
forceLoad();
}
}

@Override
protected void onStopLoading(){
		cancelLoad();
}

@Override
public void onCanceled(Cursor cursor){
		if(cursor != null && !cursor.isClosed()){
			cursor.close();
}
}

@Override
protected void onReset(){
super.onReset();
onStopLoading();
if(mCursor != null && !mCursor.isClosed()){
mCursor.close();
}
mCursor = null;
}
}
```
在RunListFragment内部实现一个SQLiteCursorLoader 的具体实现类：
```java
public class RunListFragment extends ListFragment implements LoaderCallbacks<Cursor>{
...
@Override
public void onCreate(Bundle savedInstanceState){
super.onCreate(savedInstanceState);
setHasOptionsMenu(true);
getLoaderManager().initLoader(0,null,this);  // 初始化Loader。第3个参数为一个LoaderCallbacks的实现，被LoadManager用来发送Load事件，这里是自身
第1个参数是一个ID，当调用initLoader时，如果指定ID的Loader已经存在，则返回，否则触发LoaderCallbacks的onCreateLoader()方法
}

@Override
public void onActivityResult(int requestCode, int resultCode,Intent data){
if(REQUEST_NEW_RUN == requestCode){
getLoaderManager().restartLoader(0,null,this);
}
}

@Override
public Loader<Cursor> onCreateLoader(int id,Bundle args){
return new RunListCursorLoader(getActivity());
}

@Override
public void onLoadFinished(Loader<Cursor> loader,Cursor cursor){
		RunCursorAdapter adapter = new RunCursorAdapter(getActivity(),(RunCursor)cursor);
setListAdapter(adapter);
}

@Override
public void onLoaderReset(Loader<Cursor> loader){
		setListAdapter(null);
}

private static class RunListCursorLoader extends SQLiteCursorLoader{
public RunListCursorLoader(Context context){
super(context);
}

@Override
protected Cursor loadCursor(){
return RunManager.get(getContext()).queryRuns();
}
}

}
```
