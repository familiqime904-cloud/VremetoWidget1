
package com.example.vremetowidget

import android.appwidget.AppWidgetManager
import android.appwidget.AppWidgetProvider
import android.content.Context
import android.widget.RemoteViews
import android.os.SystemClock
import java.time.LocalDateTime
import java.time.temporal.ChronoUnit

class TimeWidgetProvider : AppWidgetProvider() {

    private data class Counter(
        val title: String,
        val start: LocalDateTime,
        val calendarId: Int,
        val chronoId: Int,
        val statsId: Int
    )

    companion object {
        private val counters = listOf(
            Counter("Началото",
                LocalDateTime.of(2026, 9, 8, 18, 0),
                R.id.calendar1, R.id.chrono1, R.id.stats1),

            Counter("Забраната",
                LocalDateTime.of(2026, 11, 5, 18, 0),
                R.id.calendar2, R.id.chrono2, R.id.stats2),

            Counter("Официален край",
                LocalDateTime.of(2026, 11, 20, 18, 0),
                R.id.calendar3, R.id.chrono3, R.id.stats3)
        )

        fun updateOne(
            context: Context,
            manager: AppWidgetManager,
            widgetId: Int
        ) {
            val views = RemoteViews(
                context.packageName,
                R.layout.widget_time
            )

            val now = LocalDateTime.now()
            val nowMillis = System.currentTimeMillis()
            val elapsedRealtime = SystemClock.elapsedRealtime()

            counters.forEach { c ->
                val seconds = ChronoUnit.SECONDS
                    .between(c.start, now)
                    .coerceAtLeast(0)

                val future = now.isBefore(c.start)

                val months = if (future) 0L
                    else ChronoUnit.MONTHS.between(c.start, now)

                val anchor = c.start.plusMonths(months)

                val days = if (future) 0L
                    else ChronoUnit.DAYS.between(anchor, now)

                val calendarText = if (future) {
                    "Все още не е започнало"
                } else {
                    "$months месеца и $days дни"
                }

                views.setTextViewText(
                    c.calendarId,
                    calendarText
                )

                val startMillis = c.start
                    .atZone(java.time.ZoneId.systemDefault())
                    .toInstant().toEpochMilli()

                val elapsedMs = (nowMillis - startMillis)
                    .coerceAtLeast(0L)

                val base = elapsedRealtime - elapsedMs

                views.setChronometer(
                    c.chronoId,
                    base,
                    "%s",
                    !future
                )

                val weeks = seconds / 604800
                val totalDays = seconds / 86400
                val hours = seconds / 3600
                val minutes = seconds / 60

                val stats =
                    "Седмици $weeks · Дни $totalDays · Часове $hours\n" +
                    "Минути $minutes · Секунди $seconds"

                views.setTextViewText(c.statsId, stats)
            }

            manager.updateAppWidget(widgetId, views)
        }
    }

    override fun onUpdate(
        context: Context,
        manager: AppWidgetManager,
        ids: IntArray
    ) {
        ids.forEach {
            updateOne(context, manager, it)
        }
    }
}

package com.example.vremetowidget

import android.app.Activity
import android.appwidget.AppWidgetManager
import android.content.ComponentName
import android.os.Bundle
import android.widget.Button
import android.widget.LinearLayout
import android.widget.TextView

class MainActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val layout = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            setPadding(40, 60, 40, 40)
        }

        val title = TextView(this).apply {
            text = "Времето оттогава"
            textSize = 26f
        }

        val button = Button(this).apply {
            text = "Добави Widget"
            setOnClickListener {
                val manager =
                    AppWidgetManager.getInstance(this@MainActivity)

                val provider = ComponentName(
                    this@MainActivity,
                    TimeWidgetProvider::class.java
                )

                if (manager.isRequestPinAppWidgetSupported) {
                    manager.requestPinAppWidget(
                        provider, null, null
                    )
                }
            }
        }

        layout.addView(title)
        layout.addView(button)
        setContentView(layout)
    }
}

