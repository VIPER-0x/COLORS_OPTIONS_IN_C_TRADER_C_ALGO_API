# COLORS_OPTIONS_IN_C_TRADER_C_ALGO_API
### ترجمه و توضیح کد برای فایل `README` در GitHub به زبان فارسی

# cTrader cAlgo API: سفارشی‌سازی خطوط اندیکاتور

## مقدمه

در این راهنما، نحوه سفارشی‌سازی ظاهر خطوط رسم شده توسط اندیکاتورها در cTrader با استفاده از cAlgo API را بررسی خواهیم کرد. به طور خاص، بر روی امکان تنظیم رنگ‌ها، سبک‌ها و ضخامت‌های اختیاری خطوط توسط کاربران تمرکز خواهیم کرد.

## سفارشی‌سازی رنگ خطوط

cAlgo API شامل یک `Color` enum است که می‌تواند برای پیکربندی رنگ‌ها برای اشیاء نمودار، کنترل‌های نمودار و خروجی‌ها استفاده شود. مقادیر `Color` enum نشان‌دهنده رنگ‌های معمولاً استفاده شده هستند که می‌توانند بدون نیاز به کار با کدهای هگزادسیمال یا ARGB استفاده شوند.

### مثال: تنظیم رنگ خطوط

می‌توانید یک پارامتر از نوع `Color` تعریف کنید تا کاربران بتوانند رنگ‌های سفارشی برای خطوط رسم شده توسط اندیکاتور انتخاب کنند. در زیر مثالی از نحوه تنظیم رنگ یک خط با استفاده از یک رنگ نام‌گذاری شده و یک کد رنگ هگزادسیمال آورده شده است:

```csharp
[Output("Hexadecimal", LineColor = "#FF5733", PlotType = PlotType.Line, Thickness = 1)]
public IndicatorDataSeries Hexadecimal { get; set; }

[Output("Name", LineColor = "Red", PlotType = PlotType.Line, Thickness = 1)]
public IndicatorDataSeries Name { get; set; }
```

### امکان انتخاب رنگ‌های سفارشی توسط کاربران

برای امکان انتخاب رنگ‌های سفارشی توسط کاربران، می‌توانید یک پارامتر از نوع `Color` تعریف کنید:

```csharp
[Parameter("Drawing Color", DefaultValue = "#f54242")]
public Color DrawingColor { get; set; }
```

## سفارشی‌سازی سبک‌ها و ضخامت خطوط

علاوه بر رنگ‌ها، می‌توانید سبک‌ها و ضخامت خطوط را نیز سفارشی کنید. `LineStyle` enum شامل سبک‌های مختلفی مانند `Solid`، `Dots` و `Dash` است. می‌توانید پارامترهایی برای سبک‌ها و ضخامت خطوط تعریف کنید تا به کاربران امکان سفارشی‌سازی ظاهر خطوط را بدهید.

### مثال: تنظیم سبک‌ها و ضخامت خطوط

در زیر مثالی از نحوه تنظیم سبک‌ها و ضخامت خطوط آورده شده است:

```csharp
[Parameter("Line Style", DefaultValue = LineStyle.Solid)]
public LineStyle LineStyle { get; set; }

[Parameter("Line Thickness", DefaultValue = 1, MinValue = 1, MaxValue = 10)]
public int LineThickness { get; set; }
```

## مثال کامل پیاده‌سازی

در زیر یک پیاده‌سازی کامل از یک اندیکاتور آورده شده است که بالاترین high و پایین‌ترین low را در 256 کندل آخر تحلیل می‌کند و به کاربران امکان سفارشی‌سازی رنگ‌ها، سبک‌ها و ضخامت خطوط را می‌دهد:

```csharp name=Last256BarsHighLow.cs
using System;
using cAlgo.API;
using cAlgo.API.Internals;

namespace cAlgo
{
    [Indicator(IsOverlay = true, TimeZone = TimeZones.UTC, AccessRights = AccessRights.None)]
    public class Last256BarsHighLow : Indicator
    {
        private const int Range = 256; // تعداد کندل‌ها برای تحلیل

        [Parameter("Upper Line Color", DefaultValue = "Blue")]
        public Color UpperLineColor { get; set; }

        [Parameter("Lower Line Color", DefaultValue = "White")]
        public Color LowerLineColor { get; set; }

        [Parameter("Upper Line Style", DefaultValue = LineStyle.Solid)]
        public LineStyle UpperLineStyle { get; set; }

        [Parameter("Lower Line Style", DefaultValue = LineStyle.Solid)]
        public LineStyle LowerLineStyle { get; set; }

        [Parameter("Upper Line Thickness", DefaultValue = 1, MinValue = 1, MaxValue = 10)]
        public int UpperLineThickness { get; set; }

        [Parameter("Lower Line Thickness", DefaultValue = 1, MinValue = 1, MaxValue = 10)]
        public int LowerLineThickness { get; set; }

        protected override void Initialize()
        {
            Print("Current Chart TimeFrame: ", TimeFrame);

            int lastIndex = Bars.Count - 1;
            int firstIndex = Math.Max(0, lastIndex - Range); // اطمینان از اینکه در محدوده معتبر باقی می‌مانیم

            if (lastIndex < firstIndex)
            {
                Print("Not enough bars to analyze.");
                return;
            }

            // پیدا کردن بالاترین high و پایین‌ترین low در محدوده
            double highestHigh = double.MinValue;
            double lowestLow = double.MaxValue;
            int highestIndex = firstIndex;
            int lowestIndex = firstIndex;

            for (int i = firstIndex; i <= lastIndex; i++)
            {
                if (Bars.HighPrices[i] > highestHigh)
                {
                    highestHigh = Bars.HighPrices[i];
                    highestIndex = i;
                }

                if (Bars.LowPrices[i] < lowestLow)
                {
                    lowestLow = Bars.LowPrices[i];
                    lowestIndex = i;
                }
            }

            // گرفتن آخرین کندل در نمودار
            int chartLastIndex = Chart.BarsTotal - 1;

            // رسم خط روند برای بالاترین high
            Chart.DrawTrendLine("HighestHighLine", Bars.OpenTimes[highestIndex], highestHigh, Bars.OpenTimes[chartLastIndex], highestHigh, UpperLineColor, UpperLineThickness, UpperLineStyle);

            // رسم خط روند برای پایین‌ترین low
            Chart.DrawTrendLine("LowestLowLine", Bars.OpenTimes[lowestIndex], lowestLow, Bars.OpenTimes[chartLastIndex], lowestLow, LowerLineColor, LowerLineThickness, LowerLineStyle);

            Print($"Highest High: {highestHigh} at index {highestIndex}");
            Print($"Lowest Low: {lowestLow} at index {lowestIndex}");
        }

        public override void Calculate(int index)
        {
        }
    }
}
```



با دنبال کردن این راهنما، می‌توانید ظاهر خطوط رسم شده توسط اندیکاتورها در cTrader را با استفاده از cAlgo API سفارشی کنید. این به کاربران امکان انتخاب رنگ‌ها، سبک‌ها و ضخامت‌های سفارشی را میدهد
