<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>

    <application
        android:allowBackup="true"
        android:label="JARVIS AI"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar">

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>

        </activity>

    </application>

</manifest>
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.jarvis.ai"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.jarvis.ai"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0.0"
    }

    buildFeatures {
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.8"
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation("com.google.android.material:material:1.11.0")
    implementation("androidx.compose.ui:ui:1.6.1")
    implementation("androidx.compose.material3:material3:1.2.0")
    implementation("androidx.activity:activity-compose:1.8.2")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("org.json:json:20240303")
}
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "JarvisAI"
include(":app")
package com.jarvis.ai

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            MaterialTheme {
                Surface(modifier = Modifier.fillMaxSize()) {
                    JarvisHome()
                }
            }
        }
    }
}

@Composable
fun JarvisHome() {

    var text by remember {
        mutableStateOf("Hello, I am JARVIS.")
    }

    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {

        Text(
            text = text,
            fontSize = 28.sp
        )

        Spacer(modifier = Modifier.height(30.dp))

        Button(
            onClick = {
                text = "How may I assist you?"
            }
        ) {
            Text("Activate")
        }
    }
}
package com.jarvis.ai.ai

import okhttp3.*
import org.json.JSONArray
import org.json.JSONObject
import java.io.IOException

class AIManager(private val apiKey: String) {

    private val client = OkHttpClient()

    fun askJarvis(
        prompt: String,
        callback: (String) -> Unit
    ) {

        val json = JSONObject()

        json.put(
            "contents",
            JSONArray().put(
                JSONObject().put(
                    "parts",
                    JSONArray().put(
                        JSONObject().put("text", prompt)
                    )
                )
            )
        )

        val body = RequestBody.create(
            "application/json".toMediaType(),
            json.toString()
        )

        val request = Request.Builder()
            .url("https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$apiKey")
            .post(body)
            .build()

        client.newCall(request).enqueue(object : Callback {

            override fun onFailure(call: Call, e: IOException) {
                callback("Connection failed.")
            }

            override fun onResponse(call: Call, response: Response) {

                val result = response.body?.string()

                try {

                    val text =
                        JSONObject(result!!)
                            .getJSONArray("candidates")
                            .getJSONObject(0)
                            .getJSONObject("content")
                            .getJSONArray("parts")
                            .getJSONObject(0)
                            .getString("text")

                    callback(text)

                } catch (e: Exception) {

                    callback("Error reading AI response.")

                }

            }

        })

    }

}
package com.jarvis.ai.voice

import android.content.Context
import android.speech.tts.TextToSpeech
import java.util.Locale

class VoiceManager(context: Context) : TextToSpeech.OnInitListener {

    private var tts: TextToSpeech = TextToSpeech(context, this)
    private var ready = false

    override fun onInit(status: Int) {
        if (status == TextToSpeech.SUCCESS) {
            tts.language = Locale.US
            tts.setSpeechRate(0.9f)
            tts.setPitch(0.9f)
            ready = true
        }
    }

    fun speak(text: String) {
        if (ready) {
            tts.speak(text, TextToSpeech.QUEUE_FLUSH, null, "jarvis")
        }
    }

    fun shutdown() {
        tts.stop()
        tts.shutdown()
    }
}
package com.jarvis.ai.voice

import android.app.Activity
import android.content.Intent
import android.speech.RecognizerIntent

class SpeechRecognizerManager {

    fun startListening(activity: Activity, requestCode: Int) {

        val intent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH)

        intent.putExtra(
            RecognizerIntent.EXTRA_LANGUAGE_MODEL,
            RecognizerIntent.LANGUAGE_MODEL_FREE_FORM
        )

        intent.putExtra(
            RecognizerIntent.EXTRA_LANGUAGE,
            "en-US"
        )

        activity.startActivityForResult(intent, requestCode)
    }
}
