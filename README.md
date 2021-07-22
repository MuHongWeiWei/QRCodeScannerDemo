<h1>Android QRCode掃描器 各種條碼都能掃</h1>

<ol>
 <li><a href="https://badgameshow.com/fly/android-qr-code掃描器-各種條碼都能掃/fly/android/#a">導入Google Vision</a></li>
 <li><a href="https://badgameshow.com/fly/android-qr-code掃描器-各種條碼都能掃/fly/android/#b">聲明相機權限</a></li>
 <li><a href="https://badgameshow.com/fly/android-qr-code掃描器-各種條碼都能掃/fly/android/#c">布局&方法介紹</a></li>
 <li><a href="https://badgameshow.com/fly/android-qr-code掃描器-各種條碼都能掃/fly/android/#d">程式碼範例</a></li>
 <li><a href="https://badgameshow.com/fly/android-qr-code掃描器-各種條碼都能掃/fly/android/#e">效果展示</a></li>
 <li><a href="https://badgameshow.com/fly/android-qr-code掃描器-各種條碼都能掃/fly/android/#f">Github</a></li>

</ol>

<hr />

<a id="a"></a>

<h3>1.導入Google Vision</h3>

<h4>build.gradle(Module)</h4>

<pre><code class="language-Gradle line-numbers">dependencies {
     implementation 'com.google.android.gms:play-services-vision:20.1.3'
}
</code></pre>

<a id="b"></a>

<h3>2.聲明相機權限</h3>

```XML
<uses-permission android:name="android.permission.CAMERA"/>
```


<a id="c"></a>

<h3>3.布局&方法介紹</h3>

<h4>布局</h4>

```XML
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <SurfaceView
        android:id="@+id/scanner"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@+id/guideline2"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/result"
        android:textSize="30sp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@color/black"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/scanner" />

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.5" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

<h4>方法介紹</h4>

<h5>偵測Barcode樣式</h5>

<pre><code class="language-Kotlin line-numbers">BarcodeDetector.Builder(this@MainActivity)
        .setBarcodeFormats(Barcode.ALL_FORMATS)
        .build()
</code></pre>

<h5>創建Camera</h5>

<pre><code class="language-Kotlin line-numbers">//傳入Barcode樣式
CameraSource.Builder(this@MainActivity, barcodeDetector)
        //自動對焦
        .setAutoFocusEnabled(true)
        //預覽寬高
        .setRequestedPreviewSize(measuredWidth, measuredHeight)
        .build()
</code></pre>

<h5>綁定Camera</h5>

<pre><code class="language-Kotlin line-numbers">holder.addCallback(object : SurfaceHolder.Callback {
    override fun surfaceCreated(holder: SurfaceHolder) {
        cameraSource.start(holder)
    }

    override fun surfaceChanged(holder: SurfaceHolder, format: Int, width: Int, height: Int) {

    }

    override fun surfaceDestroyed(holder: SurfaceHolder) {
        cameraSource.stop()
    }
})
</code></pre>

<h5>掃描完成資料回傳</h5>

<pre><code class="language-Kotlin line-numbers">barcodeDetector.setProcessor(object : Detector.Processor<Barcode> {
    override fun release() {

    }

    override fun receiveDetections(detections: Detector.Detections<Barcode>) {
        result = findViewById(R.id.result)
        //獲取資料
        val barcode = detections.detectedItems
        if (barcode.size() > 0) {
            runOnUiThread {
                result.text = barcode.valueAt(0).displayValue
            }
        }
    }
})
</code></pre>

<a id="d"></a>
<h3>4.程式碼範例</h3>

<pre><code class="language-Kotlin line-numbers">package com.example.qrcodescannerdemo

import android.Manifest
import android.annotation.SuppressLint
import android.content.pm.PackageManager
import android.os.Bundle
import android.util.Log
import android.view.SurfaceHolder
import android.view.SurfaceView
import android.view.ViewTreeObserver.OnGlobalLayoutListener
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import com.google.android.gms.vision.CameraSource
import com.google.android.gms.vision.Detector
import com.google.android.gms.vision.barcode.Barcode
import com.google.android.gms.vision.barcode.BarcodeDetector

private const val REQUEST_CAMERA: Int = 40

@SuppressLint("MissingPermission")
class MainActivity : AppCompatActivity() {

    private lateinit var scanner: SurfaceView
    private lateinit var result: TextView
    private lateinit var cameraSource: CameraSource
    private lateinit var barcodeDetector: BarcodeDetector

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //移除ActionBar
        supportActionBar?.hide()

        //先獲取相機權限
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.CAMERA
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(
                this,
                arrayOf(Manifest.permission.CAMERA),
                REQUEST_CAMERA
            )
        } else {
            //初始化掃描器
            initScanner()
            //偵測後回傳
            resultCallback()
        }
    }

    private fun resultCallback() {
        barcodeDetector.setProcessor(object : Detector.Processor<Barcode> {
            override fun release() {

            }

            override fun receiveDetections(detections: Detector.Detections<Barcode>) {
                result = findViewById(R.id.result)
                //或取資料
                val barcode = detections.detectedItems
                if (barcode.size() > 0) {
                    runOnUiThread {
                        result.text = barcode.valueAt(0).displayValue
                    }
                }
            }
        })
    }

    private fun initScanner() {
        scanner = findViewById(R.id.scanner)
        scanner.apply {
            //創建Barcode偵測
            barcodeDetector = BarcodeDetector.Builder(this@MainActivity)
                .setBarcodeFormats(Barcode.ALL_FORMATS)
                .build()

            //獲取寬高後創建Camera
            viewTreeObserver.addOnGlobalLayoutListener(object : OnGlobalLayoutListener {
                override fun onGlobalLayout() {
                    viewTreeObserver.removeOnGlobalLayoutListener(this)
                    cameraSource = CameraSource.Builder(this@MainActivity, barcodeDetector)
                        .setAutoFocusEnabled(true)
                        .setRequestedPreviewSize(measuredWidth, measuredHeight)
                        .build()
                }
            })

            //camera綁定surfaceView
            holder.addCallback(object : SurfaceHolder.Callback {
                override fun surfaceCreated(holder: SurfaceHolder) {
                    cameraSource.start(holder)
                }

                override fun surfaceChanged(
                    holder: SurfaceHolder,
                    format: Int,
                    width: Int,
                    height: Int
                ) {

                }

                override fun surfaceDestroyed(holder: SurfaceHolder) {
                    cameraSource.stop()
                }
            })
        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (REQUEST_CAMERA == requestCode && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            setContentView(R.layout.activity_main)
            //初始化掃描器
            initScanner()
            //偵測後回傳
            resultCallback()
        }
    }
}
</code></pre>

<a id="e"></a>
<h4>5.效果展示</h4>

<img src="https://badgameshow.com/fly/wp-content/uploads/2021/07/1626937439782.gif" width="40%"/>

<a id="f"></a>
<h4>6.Github</h4>

<a class="wp-editor-md-post-content-link" href="https://github.com/MuHongWeiWei/QRCodeScannerDemo">Github</a>
