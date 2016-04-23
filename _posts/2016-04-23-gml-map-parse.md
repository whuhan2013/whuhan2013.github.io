---
layout: post
title: Android之解析GML并显示
date: 2016-4-23
categories: blog
tags: [地图开发]
description: Android之解析GML并显示
---   

### 本例主要实现在APP中解析GML数据并显示   

GML,地理标记语言（外语全称：Geography MarkupLanguage、外语缩写：GML），它由开放式地理信息系统协会（外语缩写：OGC）于1999年提出，并得到了许多公司的大力支持，如Oracle、Galdos、MapInfo、CubeWerx等。GML能够表示地理空间对象的空间数据和非空间属性数据 

### 实现思路   

### GML文档解析

GML文档的本质还是Xml文档，所以可以用Xml解析器进行解析，在Android中可以使用自带的XmlPullParser进行解析，当然你也可以用SAX解析器可DOM解析器。在GML文档中有两种属性，一类是空间属性，另外一个就是非空间属性。其中空间属性是至关重要的，因为要进行地图显示必须要用空间属性；而非空间属性可以不用，但是由于要进行查询操作，所以我将非空间属性也进行存储。
在读GML文档时，首先读取图层，然后判断图层中的属性都有哪些，如果是非空间属性，将其存储起来（我用的是Sqlite数据库），如果是空间属性，那么第一个空间属性肯定是关于图层类型的，即他是Polygen，Polyline或是Point，然后在<pos>或<poslist>标签下存储的位置信息，将位置信息存储起来即可。
以上就是大致的读取GML文档的思路。    


### 地图显示  

地图显示的方法有很多种，我简单介绍两种，一种是使用画布canvas进行绘制，另外一种就是使用SVG进行显示。由于我的水平有限，想不出解决canvas加载重绘时卡顿的问题，所以我就使用svg进行显示，使用svg最大的好处就是只在首次使用的时候生成SVG文件而花费的时间较长，而在之后的使用中加载时间在1s以内；另外就是使用svg进行功能实现时不会出现卡顿或者说是卡顿比前一种方法小的多。（在文件夹中包含的使用canvas绘制的工程是我找我的学长要的，哈哈）。
关于SVG大家有不明白的直接去百度，那里面比我讲的清楚的多，在写SVG的时候注意下面几个事项就行：

1. 由于GML文件里面位置数据过大，所以需要在SVG中设置ViewBox保证地图能够全部显示（关于ViewBox也去百度吧）。

2. 坐标转换，在GML中默认的坐标原点是左下角，而SVG的坐标原点是左上角，这点我想大家都因应该在之前接触过，所以需要进行坐标转换，就是进行垂直翻转而已。最初我使用的方法是将每个坐标的纵坐标变成相反数，然后就导致写SVG文件花了2分钟……。这样当然不行啊，找别的方法吧，后来终于找到个好办法，用矩阵变换，就是SVG里面的matrix属性，一下就变成10s以内，真是……

3. 图层顺序，写SVG的时候图层显示应该按照面线点文字的顺序，这样可以保证所有图层都能够显示

### 参考链接
[理解SVG坐标系统和变换： transform属性 - 推酷](http://www.tuicool.com/articles/VfyMfua) 


### 功能实现

因为我用的SVG，所以没必要再进行SVG解析，直接用浏览器就可以查看，所以我只需要实现一个浏览器的功能就行了，用WebView控件，由于SVG的放大缩小平移可以用Javascript实现，所以我的所有功能都是在Javascript中的。但是在Android里面直接加载SVG文件会导致Javascript失效，所以用html内联SVG，这个也不再多说，大家参考html文件即可。
放大缩小：放大缩小的功能就是变化ViewBox即可，大家可以参考文件夹里面的js文件。
平移：平移功能要重写WebView的ontouch事件，然后再调用Javascript函数，具体也不多说，大家自己看吧。
搜索：搜索分为两部分，首先是从数据库中按照搜索条件查询出对象的ID，然后调用Javascript搜索的相关函数搜索出SVG中对应的对象。


### WebView中使用JS代码参考链接
[Android WebView 开发详解(一) - typename 记录点滴 - 博客频道 - CSDN.NET](http://blog.csdn.net/typename/article/details/39030091)


### 实现  

第一次进入应用时，先解析GML，将数据存储在数据库中，生成SVG，用webView显示HTML来显示SVG，在里面用JS实现一些功能控制,第二次进入时直接加载SVG即可


### 主Activity  

```
package com.gmlmap.activity;

import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;

import com.gmlmap.data.LayerInfo;
import com.gmlmap.readfile.GmlPullParser;
import com.gmlmap.readfile.SvgWrite;
import com.gmlmap.util.SetViewHeight;
import com.gmlmap.util.SharedUtils;
import com.gmlmap.view.DrawerGarment;
import com.gmlmap.view.DrawerGarment.IDrawerCallbacks;

import android.annotation.SuppressLint;
import android.app.Activity;
import android.app.ProgressDialog;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.view.DragEvent;
import android.view.GestureDetector;
import android.view.GestureDetector.SimpleOnGestureListener;
import android.view.LayoutInflater;
import android.view.Menu;
import android.view.MenuItem;
import android.view.MotionEvent;
import android.view.View;
import android.view.View.OnDragListener;
import android.view.View.OnTouchListener;
import android.view.ViewGroup;
import android.view.View.OnClickListener;
import android.view.Window;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.BaseAdapter;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

public class Activity_Main extends Activity {

	private ProgressDialog dialog;
	private WebView m_webview;
	private Button m_btzoomin;
	private Button m_btzoomout;
	private Button m_btser;
	private EditText m_edit;
	private GestureDetector mGestureDetector;
	
	//抽屉使用
	private DrawerGarment mDrawerGarment;
	private TextView mButton;
	private TextView m_text;
	private ListView m_listview;
	private MAdapter m_adapter;
	private ArrayList<LayerInfo> m_layers;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		requestWindowFeature(Window.FEATURE_CUSTOM_TITLE);   
		setContentView(R.layout.activity_main);
		getWindow().setFeatureInt(Window.FEATURE_CUSTOM_TITLE, R.layout.title);  //titlebar为自己标题栏的布局
		getSvg();
		

	}

	private void InitCtView()
	{
		    mDrawerGarment = new DrawerGarment(this, R.layout.drawer);
	        m_layers=SharedUtils.getLayers(getApplicationContext(), "TestData");
	        m_text=(TextView)this.findViewById(R.id.textView1);
	        m_text.setText("图层");
	        m_listview=(ListView)this.findViewById(R.id.listview);
	        m_adapter=new MAdapter(this,m_layers);
	        m_listview.setAdapter(m_adapter);
	        SetViewHeight.setLvHeight(m_listview);
			mDrawerGarment.setDrawerCallbacks(new IDrawerCallbacks() {

				@Override
				public void onDrawerOpened() {
					
				}

				@Override
				public void onDrawerClosed() {
				}
			});
			
			mButton = (TextView) findViewById(R.id.bt_chou);
			mButton.setOnClickListener(new OnClickListener() {
				
				@Override
				public void onClick(View v) {
					mDrawerGarment.toggleDrawer();
				}
			});
	}
	
	String move="";
	
	@SuppressLint("SetJavaScriptEnabled")
	private void initwebview() {
		m_webview = (WebView) findViewById(R.id.webview);
		WebSettings webSettings = m_webview.getSettings();
		webSettings.setLoadWithOverviewMode(true);
		webSettings.setJavaScriptEnabled(true);
		m_webview.setWebViewClient(new WebViewClient());
		m_webview.setWebChromeClient(new WebChromeClient());
		// SVG图所在路径
		String svg_path="file:///mnt/sdcard/GmlParser/TestData/index.html";
		m_webview.loadUrl(svg_path);
		mGestureDetector = new GestureDetector(this, new MyOnGestureListener());
		
		m_webview.setOnTouchListener(new OnTouchListener(){

			@Override
			public boolean onTouch(View v, MotionEvent event) {
				// TODO Auto-generated method stub
				mGestureDetector.onTouchEvent(event);
				return true;
			}});
		m_edit=(EditText)this.findViewById(R.id.edittext);
		m_btzoomin=(Button)this.findViewById(R.id.mapzoomin);
		m_btzoomout=(Button)this.findViewById(R.id.mapzoomout);
		m_btser=(Button)this.findViewById(R.id.bt_search);
		m_btser.setOnClickListener(new OnClickListener(){

			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				String co=m_edit.getText().toString();
				if(co.equals(""))
					Toast.makeText(getApplicationContext(), "输入不能为空",Toast.LENGTH_SHORT).show();
				else
				{
				 String id=SharedUtils.getOid(getApplicationContext(), "TestData", co);
				 if(id.equals(""))
					Toast.makeText(getApplicationContext(), "没有搜索到结果",Toast.LENGTH_SHORT).show();
				else
					m_webview.loadUrl("javascript:searchByid('"+id+"')");
				 m_edit.setText("");
				}
			}});
		m_btzoomin.setOnClickListener(new OnClickListener(){

			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				m_webview.loadUrl("javascript:ZoomIn()");
			}});
		m_btzoomout.setOnClickListener(new OnClickListener(){

			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				m_webview.loadUrl("javascript:ZoomOut()");
			}});
	}

	private boolean GmlRead() {
		boolean result = false;
		try {
			InputStream is = getAssets().open("TestData.gml");
			result = GmlPullParser.parse(is, "TestData", this);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			result = false;
		}
		return result;
	}

	
	// Handler
	@SuppressLint("HandlerLeak")
	Handler handler = new Handler() {
		public void handleMessage(Message msg) {
			switch (msg.what) {
			case 0:
				initwebview();
				InitCtView();
				break;
			case 1:
				Toast.makeText(getApplicationContext(), "Gml解析发生错误", Toast.LENGTH_SHORT).show();
				break;
			case 2:
				Toast.makeText(getApplicationContext(), "Svg生成发生错误", Toast.LENGTH_SHORT).show();
				break;

			}
			dialog.dismiss();

		}
	};

	class mainThread implements Runnable {

		@Override
		public void run() {

			Message msg = handler.obtainMessage();
			if (GmlRead()) {
				SvgWrite m_writer = new SvgWrite(Activity_Main.this);
				if (m_writer.Write())
					msg.what = 0;// SVG生成成功
				else
					msg.what = 2;// svg生成失败
			} else
				msg.what = 1;// gml解析失败
			handler.sendMessage(msg);
		}
	}

	private void getSvg() {
		if (SharedUtils.getBooleanValue(Activity_Main.this, "isStored", false)
				&& SharedUtils.getBooleanValue(Activity_Main.this, "isSvg",
						false)) 
		{
			initwebview();
			InitCtView();
		}else {
			Toast.makeText(getApplicationContext(), "首次进入系统会花费5~20秒的时间进行数据解析，请勿退出~！",Toast.LENGTH_LONG).show();
			/**
			 * 进度条
			 */
			dialog = new ProgressDialog(Activity_Main.this,
					ProgressDialog.THEME_HOLO_LIGHT);
			dialog.setMessage("正在解析GML并生成SVG，请稍后……");
			dialog.setCanceledOnTouchOutside(false);
			dialog.show();
			Thread mainThread = new Thread(new mainThread());
			// CheckNetworkState();
			mainThread.start();
		}
	}

	public class MAdapter extends BaseAdapter {

		private ArrayList<LayerInfo> layers;
		private ArrayList<Boolean> status;
		private LayoutInflater m_listContainer; // 视图容器
		private Context context;
		
		class Views{
			CheckBox m_check;
			LinearLayout button;
		}
		
		public MAdapter(Context context,ArrayList<LayerInfo> mlayers)
		{
			this.context=context;
			layers=mlayers;
			status=new ArrayList<Boolean>();
			m_listContainer = LayoutInflater.from(context); // 创建视图容器并设置上下文
		}
		
		@Override
		public int getCount() {
			// TODO Auto-generated method stub
			return layers.size();
		}

		@Override
		public Object getItem(int arg0) {
			// TODO Auto-generated method stub
			return layers.get(arg0);
		}

		@Override
		public long getItemId(int arg0) {
			// TODO Auto-generated method stub
			return arg0;
		}

		@Override
		public View getView(int arg0, View arg1, ViewGroup arg2) {
			// TODO Auto-generated method stub
			Views m_views;
			if (arg1 == null) {
				arg1 = m_listContainer.inflate(R.layout.listdetail,
						null);
				m_views=new Views();
				m_views.m_check = (CheckBox) arg1.findViewById(R.id.checkbox);
				//m_views.button=(LinearLayout)arg1.findViewById(R.id.bt_attr);
				m_views.m_check.setText(layers.get(arg0).Lname);
				m_views.m_check.setTag(arg0);
				//m_views.button.setTag(layers.get(arg0).Lid);
				status.add(false);
				arg1.setTag(m_views);
			} else {
				m_views = (Views) arg1.getTag();
			}
			m_views.m_check.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	            	m_webview.loadUrl("javascript:changeVisible('L_"+v.getTag()+"')");
	            }
	        });
			/*m_views.button.setOnClickListener(new OnClickListener(){

				@Override
				public void onClick(View v) {
					// TODO Auto-generated method stub
					Intent intent=new Intent(Activity_Main.this,Activity_attr.class);
					intent.putExtra("name", "L_"+v.getTag());
					startActivity(intent);
				}});*/
			
			return arg1;
		}

	}
	
	class MyOnGestureListener extends SimpleOnGestureListener{
	       //滑动    Drag
		   @Override
	        public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
			   m_webview.loadUrl("javascript:scrool("+distanceX+","+distanceY+")");
			return false;
	        }

		   @Override
	        public boolean onDoubleTap(MotionEvent e) {
			   m_webview.loadUrl("javascript:ZoomIn()");
	            return false;
	        }
	        //抬起
	        @Override
	        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
	        	
	        	return false;
	        }

	        @Override
	        public void onShowPress(MotionEvent e) {
	        }

	        @Override
	        public boolean onDown(MotionEvent e) {
	        	
	        	return false;
	        }
	}
	
}
```  


### 首先解析GML  

```
package com.gmlmap.readfile;

import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;

import org.xmlpull.v1.XmlPullParser;
import org.xmlpull.v1.XmlPullParserException;

import com.gmlmap.data.LayerInfo;
import com.gmlmap.data.PosInfo;
import com.gmlmap.util.SharedUtils;

import android.content.Context;
import android.content.SharedPreferences;
import android.content.SharedPreferences.Editor;
import android.database.sqlite.SQLiteDatabase;
import android.util.Xml;

public class GmlPullParser {
	
	public static String GML_PREFIX="http://www.opengis.net/gml" ;
	public final static int CASE_NORMAL=0;     //一般情况，不进行任何操作，直接跳
	public final static int CASE_FIRSTREAD=1;  //第一次读featureMember，即读取图层的第一个要素
	public final static int CASE_READ=2;       //读取图层的第二个以后的要素
	public final static int CASE_READLNAME=3;  //读取图层名称
	public static String SQL_NORMAL_INSERT="insert into";
	public static String SQL_NORMAL_CREATE="CREATE TABLE";
	
	public static boolean parse(InputStream is,String filename,Context context){
	 
		boolean result=false;
		SharedUtils.StorePreferences(context, "filename", filename);
		ArrayList<String> AttrNames=new ArrayList<String>();
		LayerInfo m_layerinfo=new LayerInfo();
		PosInfo m_posinfo=new PosInfo();
		
		int m_curCondition=CASE_NORMAL;//当前位置
		
		//打开或创建数据库  
        SQLiteDatabase db = context.openOrCreateDatabase(filename+".db", Context.MODE_PRIVATE, null); 
        db.beginTransaction();
        db.execSQL("DROP TABLE IF EXISTS LayersInfo");  
        db.execSQL("create table LayersInfo (Lid INTEGER  primary key,Lname VARCHAR(20),Ltype INTEGER)");
        db.execSQL("DROP TABLE IF EXISTS FeaturePositon");  
        db.execSQL("create table FeaturePositon (Pid INTEGER  primary key,Lid INTEGER,OBJECTID INTEGER,Pos TEXT)");
        //创建xml pull解析器
        XmlPullParser parser = Xml.newPullParser();
		try {
			parser.setInput(is, "UTF-8");
		
		
		int eventType = parser.getEventType();
		
		while (eventType!=XmlPullParser.END_DOCUMENT) {
			switch (eventType) {
			
			case XmlPullParser.START_DOCUMENT:
				break;
			
			case XmlPullParser.START_TAG:
				if (parser.getName().equals("featureMember")) {
					m_curCondition=CASE_READLNAME;
					}
				else if(parser.getName().equals("lowerCorner"))
				{
					eventType=parser.next();
					SharedUtils.StorePreferences(context,"lowerCorner",parser.getText());
				}
				else if(parser.getName().equals("upperCorner"))
				{
					eventType=parser.next();
					SharedUtils.StorePreferences(context,"upperCorner",parser.getText());
				}
				else if(m_curCondition!=CASE_NORMAL && parser.getNamespace().equals(GML_PREFIX))
			    {
					m_layerinfo.Ltype=getLayerType(parser.getName());
					eventType = parser.next();
					while (eventType!=XmlPullParser.END_DOCUMENT) 
					{
						if(parser.getName()!=null && parser.getName().contains("pos"))
						{
							eventType = parser.next();
							m_posinfo.Pos=parser.getText();
							db.execSQL(m_posinfo.insertData());
							m_posinfo.Pid++;
							break;
						}
						else
							eventType = parser.next();
					}
			    } 
				else
				{
					switch(m_curCondition)
					{
					case CASE_READLNAME:
						if(!parser.getName().equals(m_layerinfo.Lname))
						{
							m_layerinfo.Lid++;
							m_layerinfo.Ltype=-1;
							m_posinfo.Lid=m_layerinfo.Lid;
							m_layerinfo.Lname=parser.getName();
							m_curCondition=CASE_FIRSTREAD;
							AttrNames.removeAll(AttrNames);
						}
						else
							m_curCondition=CASE_READ;
						m_layerinfo.InitInfo();
						break;
						
					case CASE_FIRSTREAD:
						boolean ifObj=false;
						m_layerinfo.AddInsertName(parser.getName());
				    	AttrNames.add(parser.getName());
						if(parser.getName().equals("OBJECTID"))
							ifObj=true;
						else
							m_layerinfo.AddCreate(parser.getName());
						eventType = parser.next();
						m_layerinfo.AddInsertValue(parser.getText());
						if(ifObj)
							m_posinfo.OBJECTID=Integer.parseInt(parser.getText());
						break;
						
					case CASE_READ:
						if(AttrNames.contains(parser.getName()))
						{
							boolean ifObj2=false;
							if(parser.getName().equals("OBJECTID"))
								ifObj2=true;
					    	m_layerinfo.AddInsertName(parser.getName());
					    	eventType = parser.next();
					    	m_layerinfo.AddInsertValue(parser.getText());
					    	if(ifObj2)
					    		m_posinfo.OBJECTID=Integer.parseInt(parser.getText());
						}
						break;
					}
				}
				break;
				
			case XmlPullParser.END_TAG:
				
				if (parser.getName().equals("featureMember")) {
					if(m_curCondition==CASE_FIRSTREAD)
					{
						db.execSQL("DROP TABLE IF EXISTS L_"+m_layerinfo.Lid);
						db.execSQL(m_layerinfo.getCreateInfo());
						db.execSQL(m_layerinfo.insertData());
					}
				  	db.execSQL(m_layerinfo.getInsertInfo());
				}
				
				break;
				
			case XmlPullParser.END_DOCUMENT:
				break;
			}
			eventType = parser.next();
			}
		//关闭当前数据库 
		 db.setTransactionSuccessful(); 
	     db.endTransaction(); 
         db.close(); 
         result=true;
        SharedUtils.StorePreferences(context, "isStored", true);
		} catch (XmlPullParserException e) {
			// TODO Auto-generated catch block
			db.close();
			result=false;
		} catch (IOException e) {
			// TODO Auto-generated catch block
			db.close();
			result=false;
		} 
		return result;
	}
	
	/*
	 * 获取图层的类型，-1表示非空间信息，0表示Point，1表示PolyLine，2表示PolyGen
	 */
	private static int getLayerType(String name)
	{
		name=name.toLowerCase();
		int result=-1;
		if(name.contains("point"))
			result=0;
		else if(name.contains("curve") || name.contains("line") || name.contains("polyline"))
			result=1;
		else if(name.contains("surface") || name.contains("polygon"))
			result=2;
		return result;
	}
	
	
}
```   


### 生成SVG  

```
package com.gmlmap.readfile;

import java.io.BufferedWriter;
import java.io.File;
import java.io.FileOutputStream;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

import com.gmlmap.activity.Activity_Main;
import com.gmlmap.data.LayerInfo;
import com.gmlmap.util.SharedUtils;
import com.gmlmap.util.SvgUtils;

import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Environment;
import android.widget.Toast;

public class SvgWrite {

	private Context context;
	private StringBuilder m_buffer = null;
	private List<LayerInfo> m_layerinfos;
	private String folderName = "";

	public SvgWrite(Context context) {
		this.context = context;
	}

	/*
	 * 创建文件夹
	 */
	private void createFolder(String foldername) {
		String pathUrl = Environment.getExternalStorageDirectory().getPath()
				+ "/GmlParser/" + foldername;
		File file = new File(pathUrl);
		if (!file.exists())
			file.mkdirs();
		copy("htmfuc.js", pathUrl);
		copy("index.html", pathUrl);
	}

	/**
	 * 
	 * @param ASSETS_NAME
	 *            要复制的文件名
	 * @param savePath
	 *            要保存的路径 testCopy(Context context)是一个测试例子。
	 */

	private void copy(String ASSETS_NAME, String savePath) {
		String filename = savePath + "/" + ASSETS_NAME;

		File dir = new File(savePath);
		// 如果目录不中存在，创建这个目录
		if (!dir.exists())
			dir.mkdir();
		try {
			if (!(new File(filename)).exists()) {
				InputStream is = context.getResources().getAssets()
						.open(ASSETS_NAME);
				FileOutputStream fos = new FileOutputStream(filename);
				byte[] buffer = new byte[7168];
				int count = 0;
				while ((count = is.read(buffer)) > 0) {
					fos.write(buffer, 0, count);
				}
				fos.close();
				is.close();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	public void deleteFile(File file) {
		if (file.exists()) { // 判断文件是否存在
		if (file.isFile()) { // 判断是否是文件
		file.delete(); // delete()方法 你应该知道 是删除的意思;
		
		}
		} 
		}
	
	/*
	 * 创建文件
	 */
	public boolean createFile(String foldername) {
		boolean result = false;
		File fileRoot = new File(Environment.getExternalStorageDirectory()
				.getPath() + "/GmlParser");
		if (!fileRoot.exists()) {
			fileRoot.mkdir();
		}
		createFolder(foldername);
		File file = new File(Environment.getExternalStorageDirectory()
				.getPath() + "/GmlParser/" + foldername + "/map.svg");
		if (file.exists()) 
			file.delete();
			try {
				file.createNewFile();
				result = true;
			} catch (IOException e) {
				// TODO Auto-generated catch block
				result = false;
			} 
		return result;
	}

	public Boolean Write() {
		boolean result = false;
		SvgUtils m_svg = new SvgUtils(context);
		AddContent(m_svg.getDocumentStart());
		folderName = SharedUtils
				.getStringValue(context, "filename", "TestData");
		// 打开或创建数据库
		SQLiteDatabase db = context.openOrCreateDatabase(folderName + ".db",
				Context.MODE_PRIVATE, null);
		if (m_layerinfos == null)
			m_layerinfos = new ArrayList<LayerInfo>();
		Cursor c = db.rawQuery(getLinfoSelect(), null);
		while (c.moveToNext()) {

			LayerInfo ainfo = new LayerInfo();
			ainfo.Lid = c.getInt(c.getColumnIndex("Lid"));
			ainfo.Lname = c.getString(c.getColumnIndex("Lname"));
			ainfo.Ltype = c.getInt(c.getColumnIndex("Ltype"));
			Random random = new Random();
			ainfo.Lstyle = random.nextInt(3);
			m_layerinfos.add(ainfo);
		}
		c.close();

		for (int i = 0; i < m_layerinfos.size(); i++) {
			int Lid = m_layerinfos.get(i).Lid;
			if (m_layerinfos.get(i).Ltype != -1) {
				AddContent(m_svg.SetLayerStart(Lid));
				c = db.rawQuery(getPinfoSelect(Lid), null);
				while (c.moveToNext()) {
					String PosList = c.getString(c.getColumnIndex("Pos"));
					AddContent(m_svg.Draw(PosList, m_layerinfos.get(i).Ltype,
							m_layerinfos.get(i).Lstyle));
				}
				c.close();
			}
			else
			{
				AddContent("</g>"+m_svg.SetLayerStart(Lid));
				c = db.rawQuery(getTinfoSelect(Lid), null);
				while (c.moveToNext()) {
					String PosList = c.getString(c.getColumnIndex("Pos"));
					String Name=c.getString(c.getColumnIndex("Name"));
					String Oid="O_"+c.getInt(c.getColumnIndex("Oid"));
					AddContent(m_svg.DrawText(PosList, Name,Oid));
				}
				c.close();
			}
			AddContent(SvgUtils.SVG_LAYER_ENDTAG);
		}
		AddContent(SvgUtils.SVG_LAYER_ENDTAG + SvgUtils.SVG_ENDDOUCUMENT);
		result = WriteContent(this.m_buffer.toString());
		if (result)
			SharedUtils.StorePreferences(context, "isSvg", result);
		return result;
	}

	private void AddContent(String str) {
		if (m_buffer == null)
			m_buffer = new StringBuilder();
		
		m_buffer.append(str);
	}

	// 向已创建的文件中写入数据库中存储的数据
	private boolean WriteContent(String str) {
		boolean result = false;
		try {
			createFile(folderName);
			FileWriter fw = null;
			BufferedWriter bw = null;
			fw = new FileWriter(Environment.getExternalStorageDirectory()
					.getPath() + "/GmlParser/" + folderName + "/map.svg", true);//
			// 创建FileWriter对象，用来写入字符流
			bw = new BufferedWriter(fw); // 将缓冲对文件的输出
			bw.write(str); // 写入文件
			bw.newLine();
			bw.flush(); // 刷新该流的缓冲
			bw.close();
			fw.close();
			result = true;

		} catch (IOException e) {
			// TODO Auto-generated catch block
			result = false;
		}
		return result;
	}

	/*
	 * 获取图层信息的select语句
	 */
	private String getLinfoSelect() {
		return "select * from LayersInfo order by Ltype DESC";
	}

	private String getPinfoSelect(int Lid) {
		return "select Pos from FeaturePositon where Lid=" + Lid;
	}
	
	//文字
	private String getTinfoSelect(int Lid)
	{
		//String result="select name.TextString Name,pos.Pos Pos from ";
		//String name="(Select FeatureId,TextString From L_"+Lid+" ) name,FeaturePositon pos";
	    //return result+name+"where name.FeatureId=pos.OBJECTID";
		return "select name.TextString Name,name.OBJECTID Oid,pos.Pos Pos from (Select OBJECTID,FeatureId,TextString From L_4 ) name,FeaturePositon pos where name.FeatureId=pos.OBJECTID";
	}
}
```  


### 其中要用到SvgUtils  

```
package com.gmlmap.util;

import java.util.Random;

import android.content.Context;

public class SvgUtils {
	
	public static String[] PolygenColors={"#dfeafc","#faf3df","#fce3e3","#fed4cc"};
	public static String[] PolylineColors={"#0c343d","#274e13","#783f04","#073763"};
	public static String[] PointColors={"#ff0000","#ff9900","#00ff00","#ffff00"};
	public static String[] Stroks={"#330000","#003300","#660000","#000080","#FFD700"};
	
	public static String XML_BASE="<?xml version=\"1.0\" encoding=\"utf-8\"?>";
	public static String SVG_BASE="<svg id=\"root\"  xmlns=\"http://www.w3.org/2000/svg\"  xmlns:xlink=\"http://www.w3.org/1999/xlink\" ";
	public static String SVG_LAYERS="<g id=\"layers\"><g  transform=\"matrix(1,0,0,-1,0,0)\">";
	public static String SVG_LAYER_STARTTAG="<g id=\"L_";
    public static String SVG_LAYER_ENDTAG="</g>";
    public static String SVG_ENDDOUCUMENT="</svg>";
    private double LOWER_X;
    private double LOWER_Y;
    private double UPPER_X;
    private double UPPER_Y;
    
    private Context m_context;
    public SvgUtils(Context context)
    {
    	this.m_context=context;
    	SharedUtils.StorePreferences(context,"lowerCorner","585305.346019984 3338389.98497472");
    	SharedUtils.StorePreferences(context,"upperCorner","596424.775248615 3345263.17995185");
    	String lowerCorner=SharedUtils.getStringValue(m_context, "lowerCorner", "0 0");
    	String upperCorner=SharedUtils.getStringValue(m_context, "upperCorner", "1000 1000");
    	String poslist1[]=lowerCorner.split(" ");
    	String poslist2[]=upperCorner.split(" ");
    	LOWER_X=Double.parseDouble(poslist1[0]);
    	LOWER_Y=Double.parseDouble(poslist1[1]);
    	UPPER_X=Double.parseDouble(poslist2[0]);
    	UPPER_Y=Double.parseDouble(poslist2[1]);}
    
    public String getDocumentStart()
    {
    	return XML_BASE+SVG_BASE+" width=\"100%\" height=\"100%\" viewBox=\""+LOWER_X+",-"+UPPER_Y+","+(UPPER_X-LOWER_X)+","+(UPPER_Y-LOWER_Y)
    			+"\">"+SVG_LAYERS;
    }
    
   
    public String SetLayerStart(int Lid)
    {
    	return SVG_LAYER_STARTTAG+Lid+"\">";
    }
	
    public String Draw(String PosList,int Ltype,int Lstyle)
    {
    	String result="";
    	switch(Ltype)
    	{
    	case 0:
    		result=DrawPoint(PosList,PointColors[Lstyle]);
    		break;
    	case 1:
    		result=DrawPolyLine(PosList,PolylineColors[Lstyle]);
    		break;
    	case 2:
    	{
    		Random random = new Random();
    		
    		result=DrawPolyGen(PosList,PolygenColors[Lstyle],Stroks[ random.nextInt(4)]);
    		
    	}
    		break;
    	}
    	return result;
    }
    
    
    //根据点集确定text位置
    private String getTextPos(String Pos)
    {
    	String result="";
    	double lowerx=0,upperx=0,lowery=0,uppery=0;
    	String[] poslist=Pos.split(" ");
    	for(int i=0;i<poslist.length;i++)
    	{
    		//x? y:z
    	  if(i%2==0)
    	  {
    		  double x=Double.parseDouble(poslist[i]);
    		  if(i==0)
    		     lowerx=x;
    		  else
    			  lowerx=(lowerx>x)?x:lowerx;
    		  upperx=(upperx<x)?x:upperx;
    	  }
    	  else
    	  {
    		  double y=Double.parseDouble(poslist[i]);
    		  if(i==1)
    			  lowery=y;
    		  else
    			  lowery=(lowery>y)?y:lowery;
    		  uppery=(uppery<y)?y:uppery;
    	  }
    		
    	}
    	result="x=\""+(lowerx+(upperx-lowerx)/4)+"\" y=\"-"+((uppery)-(uppery-lowery)/2)+"\" ";
    	return result;
    }
    
    public String DrawText(String Pos,String name,String Oid)
    {
    	String result="";
    	result="<text id=\""+Oid+"\" "+getTextPos(Pos)+"fill=\"black\" font-size=\"260\">"+name+"</text>";
    	return result;
    }
    
    public String DrawPoint(String Pos,String Color)
    {
    	String Poss[]=Pos.split(" ");
    	String result="<circle cx=\""+Poss[0]+"\" cy=\""+Poss[1]+"\" r=\"15\" fill=\""+Color+"\"/>";
    	return result;
    }

    public String DrawPolyLine(String Pos,String lineColor)
    {
    	String result="<polyline points=\" "+Pos+"\" style=\"fill:none;stroke:"+lineColor+";stroke-width:1.5\" />";
    	return result;
    }
    
    public String DrawPolyGen(String Pos,String fillColor,String strokeColor)
    {
    	String result="<polygon points=\""+Pos+"\" style=\"fill:"+fillColor+";stroke:"+strokeColor+";stroke-width:1.5\"/>";
    	return result;
    }
    
    /*
     * 坐标转换
     */
    private String convertCoorX(String x)
    {
    	return Double.parseDouble(x)-LOWER_X+"";
    }
    private String convertCoorY(String y)
    {
    	return Double.parseDouble(y)-LOWER_Y+"";
    }
}
```   


### HTML文件

```
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=gb2312">
<title> map </title>
<script src="htmfuc.js"  type="text/javascript" ></script>
<style> 
.div-a{ position:absolute; left:0px; top:0px;background-color:transparent;width:100%; height:100%  } 
</style> 
</head>

<body onload="Init()">
<div class="div-a" filter:alpha(opacity=100,finishopacity=0,style=2></div> 
<embed id="map" src="./map.svg" left="0px" top="0px"; width="100%" height="100%" type="image/svg+xml" >
</body>
</html>
```   


### JS文件控制

```
var htmlObj, SVGDoc, SVGRoot, viewBox, goLeft, goRight, innerSVG;
var currentSize, currentPosition, currentRoomId, currentRoomLabel;
var svgns = "http://www.w3.org/2000/svg";
var S_Width,S_Height;
var curtype=0;

function Init()
{

  htmlObj = document.getElementById("map");
   SVGDoc = htmlObj.getSVGDocument();
   SVGRoot = SVGDoc.documentElement;
   S_Width=screen.width;
   S_Height=screen.height;
}


function changeVisible(tagname){

    var tag=SVGDoc.getElementById(tagname);
	
	if(!tag)
	{
		return;
	}
	var visible=tag.getAttribute("visibility");
	
	if(visible=="hidden")
	{
		tag.setAttribute("visibility","visible");
	}
	else
	{
		tag.setAttribute("visibility","hidden");  
	}

} 

function ZoomIn()
{

if(curtype<5)
{
curtype=curtype+1;
   var rootElement=SVGDoc.getElementById("root");
   var viewBox = rootElement.getAttribute('viewBox');	// Grab the object representing the SVG element's viewBox attribute.

   var viewBoxValues = viewBox.split(',');
   viewBoxValues[0] = parseFloat(viewBoxValues[0]);	// LeftX
   viewBoxValues[1] = parseFloat(viewBoxValues[1]);	// UpY
   viewBoxValues[2] = parseFloat(viewBoxValues[2]);	// UWdith
viewBoxValues[3] = parseFloat(viewBoxValues[3]);
   viewBoxValues[0]=getLower(viewBoxValues[0],viewBoxValues[2],viewBoxValues[2]/2);
   viewBoxValues[1]=getLower(viewBoxValues[1],viewBoxValues[3],viewBoxValues[3]/2);
   viewBoxValues[2] = viewBoxValues[2]/2;
viewBoxValues[3] = viewBoxValues[3]/2;   
rootElement.setAttribute('viewBox', viewBoxValues.join(','));
}
}

function ZoomOut()
{
if(curtype>-2)
{
curtype=curtype-1;
   var rootElement=SVGDoc.getElementById("root");
   var viewBox = rootElement.getAttribute('viewBox');	// Grab the object representing the SVG element's viewBox attribute.

   var viewBoxValues = viewBox.split(',');
   viewBoxValues[0] = parseFloat(viewBoxValues[0]);	// LeftX
   viewBoxValues[1] = parseFloat(viewBoxValues[1]);	// UpY
   viewBoxValues[2] = parseFloat(viewBoxValues[2]);	// UWdith
   viewBoxValues[3] = parseFloat(viewBoxValues[3]);
   viewBoxValues[0]=getLower(viewBoxValues[0],viewBoxValues[2],viewBoxValues[2]*2);
   viewBoxValues[1]=getLower(viewBoxValues[1],viewBoxValues[3],viewBoxValues[3]*2);
   viewBoxValues[2] = viewBoxValues[2]/0.5;
viewBoxValues[3] = viewBoxValues[3]/0.5;   
rootElement.setAttribute('viewBox', viewBoxValues.join(','));
}
}

function getLower(oldLower,oldValue,newValue)
{
  return oldLower+oldValue/2-newValue/2;
}

function searchByid(tname)
{
   var oElement=SVGDoc.getElementById(tname);
   var x=parseFloat(oElement.getAttribute('x'));
   var y=parseFloat(oElement.getAttribute('y'));
   var rootElement=SVGDoc.getElementById("root"); 
   var viewBox = rootElement.getAttribute('viewBox');	// Grab the object representing the SVG element's viewBox attribute.
   var viewBoxValues = viewBox.split(',');
   viewBoxValues[2]=parseFloat(viewBoxValues[2]);
   viewBoxValues[3]=parseFloat(viewBoxValues[3]);
   viewBoxValues[0]=x-viewBoxValues[2]/2;
   viewBoxValues[1]=y-viewBoxValues[3]/2;
   rootElement.setAttribute('viewBox', viewBoxValues.join(','));
}

function Pan()
{
   var panRate    = 10;
   var rootElement=SVGDoc.getElementById("root");
   var viewBox = rootElement.getAttribute('viewBox');	// Grab the object representing the SVG element's viewBox attribute.
   var viewBoxValues = viewBox.split(',');
   viewBoxValues[0] = parseFloat(viewBoxValues[0]);	// LeftX
   viewBoxValues[1] = parseFloat(viewBoxValues[1]);	// UpY
   viewBoxValues[2] = parseFloat(viewBoxValues[2]);	// UWdith
   viewBoxValues[0] += panRate*viewBoxValues[2] /S_Width;
   rootElement.setAttribute('viewBox', viewBoxValues.join(','));
}

function scrool(distanceX,distanceY)
{
   var rootElement=SVGDoc.getElementById("root");
   var viewBox = rootElement.getAttribute('viewBox');	// Grab the object representing the SVG element's viewBox attribute.
   var viewBoxValues = viewBox.split(',');
   viewBoxValues[0] = parseFloat(viewBoxValues[0]);	// LeftX
   viewBoxValues[1] = parseFloat(viewBoxValues[1]);	// UpY
   viewBoxValues[2] = parseFloat(viewBoxValues[2]);	// UWdith
   viewBoxValues[3] = parseFloat(viewBoxValues[3]);	// Uheight 
   viewBoxValues[0] += parseFloat(distanceX)*viewBoxValues[2] /S_Width;
   viewBoxValues[1] += parseFloat(distanceY)*viewBoxValues[3] /S_Height;
   rootElement.setAttribute('viewBox', viewBoxValues.join(','));
  
}

function Zoom(x,y)
{
 if ( evt.detail ==2 ){
           var panRate    = 10;
   var rootElement=SVGDoc.getElementById("root");
   var viewBox = rootElement.getAttribute('viewBox');	// Grab the object representing the SVG element's viewBox attribute.

   var viewBoxValues = viewBox.split(',');
   x=parseFloat(x);
   y=parseFloat(y);
   viewBoxValues[0] = parseFloat(viewBoxValues[0]);	// LeftX
   viewBoxValues[1] = parseFloat(viewBoxValues[1]);	// UpY
   viewBoxValues[2] = parseFloat(viewBoxValues[2]);	// UWdith
   viewBoxValues[3] = parseFloat(viewBoxValues[3]);
   viewBoxValues[0]= getBasecor(getTruecorX(x,viewBoxValues[0],viewBoxValues[2] ),viewBoxValues[2]/2);
   viewBoxValues[1]= getBasecor(getTruecorY(y,viewBoxValues[1],viewBoxValues[3] ),viewBoxValues[3]/2);   
   viewBoxValues[2] = viewBoxValues[2]/2;
   viewBoxValues[3] = viewBoxValues[3]/2;   
rootElement.setAttribute('viewBox', viewBoxValues.join(','));
         }
}


function getTruecorX(CorX,LowerX,Width)
{
   return  LowerX+Width/S_Width*CorX;
}

function getTruecorY(CorY,LowerY,Height)
{
   return  LowerY+Height/S_Height*CorY;
}

function getBasecor(Cor,value)
{
   return  Cor-value/2;
}
```   


### 参考链接
[Gml解析并显示（Android） - 下载频道 - CSDN.NET](http://download.csdn.net/detail/kevingarnet/8896421)

### 完成，效果如下  


![这里写图片描述](http://img.blog.csdn.net/20160423192845408)
























