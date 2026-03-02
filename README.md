package com.example.prosketch

import android.content.Context
import android.graphics.*
import android.os.Bundle
import android.view.MotionEvent
import android.view.View
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.viewinterop.AndroidView

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            DrawingScreen()
        }
    }
}

@Composable
fun DrawingScreen() {
    AndroidView(
        modifier = Modifier.fillMaxSize(),
        factory = { context ->
            DrawingView(context)
        }
    )
}

class DrawingView(context: Context) : View(context) {

    private val drawPaint = Paint().apply {
        color = Color.BLACK
        isAntiAlias = true
        strokeWidth = 8f
        style = Paint.Style.STROKE
        strokeCap = Paint.Cap.ROUND
        strokeJoin = Paint.Join.ROUND
    }

    private val drawPath = Path()
    private var lastX = 0f
    private var lastY = 0f

    override fun onTouchEvent(event: MotionEvent): Boolean {

        // Stylus pressure support
        val pressure = event.pressure.coerceIn(0.1f, 1f)
        drawPaint.strokeWidth = pressure * 25f

        val x = event.x
        val y = event.y

        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                drawPath.moveTo(x, y)
                lastX = x
                lastY = y
            }

            MotionEvent.ACTION_MOVE -> {
                val dx = Math.abs(x - lastX)
                val dy = Math.abs(y - lastY)

                if (dx >= 4f || dy >= 4f) {
                    drawPath.quadTo(
                        lastX,
                        lastY,
                        (x + lastX) / 2,
                        (y + lastY) / 2
                    )
                    lastX = x
                    lastY = y
                }
            }

            MotionEvent.ACTION_UP -> {
                drawPath.lineTo(lastX, lastY)
            }
        }

        invalidate()
        return true
    }

    override fun onDraw(canvas: Canvas) {
        canvas.drawColor(Color.WHITE)
        canvas.drawPath(drawPath, drawPaint)
    }
}
