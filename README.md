#WaveTrend 4h/24m

//@version=5
indicator("WaveTrend 4h/24m")

// inputs
wtShow = input.bool(true, title = 'Show WaveTrend')
vwapShow = input.bool(false, title= "Show VWAP 1")
vwapShow2 = input.bool(false, title= "Show VWAP 2")

tf1 = input.timeframe(title="WaveTrend Timeframe 1", defval="24", tooltip= "Add chart time intervals for more timeframe options in this dropdown menu", group= "Timeframes")
tf2 = input.timeframe(title="WaveTrend Timeframe 2", defval="240", tooltip= "Add chart time intervals for more timeframe options in this dropdown menu", group= "Timeframes")
tf1I = input.timeframe(title="VWAP Timeframe 1", defval="24", tooltip= "Add chart time intervals for more timeframe options in this dropdown menu", group= "Timeframes")
tf2I = input.timeframe(title="VWAP Timeframe 2", defval="240", tooltip= "Add chart time intervals for more timeframe options in this dropdown menu", group= "Timeframes")


// Wavetrend calculations 
n1 = 9
n2 = 12
average = "None"
av_period = 3

ap = hlc3 
esa = ta.ema(ap, n1)
d = ta.ema(math.abs(ap - esa), n1)
ci = (ap - esa) / (0.015 * d)
tci = ta.ema(ci, n2)

wt1 = tci
wt2 = ta.sma(wt1,3)
wt3 = wt1 - wt2

//Multiple Timeframes
tf1vwap = request.security(syminfo.tickerid, tf1I, wt3, lookahead=barmerge.lookahead_off)
tf2vwap = request.security(syminfo.tickerid, tf2I, wt3, lookahead=barmerge.lookahead_off)


tf1WaveTrend1 = request.security(syminfo.tickerid, tf1, wt1)
tf1WaveTrend2 = request.security(syminfo.tickerid, tf1, wt2)
tf2WaveTrend1 = request.security(syminfo.tickerid, tf2, wt1)
tf2WaveTrend2 = request.security(syminfo.tickerid, tf2, wt2)
    
// horizontal lines
obLevel = 60
trigger1 = 53
osLevel = -60
trigger2 = -53

//Wavetrend Cross
wtCross1 = ta.cross(tf1WaveTrend1, tf1WaveTrend2)
wtCross2 = ta.cross(tf2WaveTrend1, tf2WaveTrend2)
signalColor1 = tf1WaveTrend2 - tf1WaveTrend1 > 0 ? color.red : color.lime
signalColor2 = tf2WaveTrend2 - tf2WaveTrend1 > 0 ? color.red : color.lime

//plots
hline(0, title="Zero Line", color=color.white, linestyle=hline.style_dotted)
plot(wtShow ? tf1WaveTrend1 : na, style = plot.style_area, title = 'WT Wave 1', color = #90caf9, transp = 25)
plot(wtShow ? tf1WaveTrend2 : na, style = plot.style_area, title = 'WT Wave 2', color = #0d47a1, transp = 20)
wt1Plot = plot(wtShow ? tf2WaveTrend1 : na, style = plot.style_line, linewidth= 2, title = 'WT Wave 1', color = #21baf3, transp = 25)
wt2Plot = plot(wtShow ? tf2WaveTrend2 : na, style = plot.style_line, linewidth = 2, title = 'WT Wave 2', color =#673ab7, transp = 20)
plot(vwapShow ? tf1vwap : na, style = plot.style_area, title = 'VWAP 1', color = color.yellow, transp = 30)
plot(vwapShow2 ? tf2vwap : na, style = plot.style_area, title = 'VWAP 2', color = color.orange, transp = 60)

plot(wtCross1 ? tf1WaveTrend2 : na, title="buy/sell circles", color = signalColor1, style=plot.style_circles, linewidth = 3)
plot(wtCross2 ? tf2WaveTrend2 : na, title="buy/sell circles", color = signalColor2, style=plot.style_circles, linewidth = 3)
plot(obLevel, style = plot.style_line, title = "Over Bought level 1", color=color.white, transp = 20)
plot(osLevel, style = plot.style_line, title = "Over Sold level 1", color=color.white, transp = 20)
plot(trigger1, style = plot.style_circles, title = "Tigger 1", color=color.white, transp = 50)
plot(trigger2, style = plot.style_circles, title = "Tigger 2", color=color.white, transp = 50)

waveFillColor = tf2WaveTrend1 >= tf2WaveTrend2 ? color.new(#21baf3, 75) : color.new(#673ab7, 60)

fill(wt1Plot, wt2Plot, title="WaveTrend Fill", color = waveFillColor)

allVwapOver = ((tf1vwap > 0) and (ta.crossover(tf2vwap, 0))) or ((ta.crossover(tf1vwap, 0) and (tf2vwap > 0)))
allVwapUnder = ((tf1vwap < 0) and (ta.crossunder(tf2vwap, 0))) or ((ta.crossunder(tf1vwap, 0) and (tf2vwap < 0)))

plotshape(allVwapOver, title='Vwaps above 0', location=location.bottom, style=shape.diamond, size=size.tiny, color=#2C9670, transp=25)
plotshape(allVwapUnder, title='Vwaps under 0', location=location.top, style=shape.diamond, size=size.tiny, color=#B84343, transp=25)

alertcondition(allVwapOver, title="Long Entry", message="Long Entry")
alertcondition(allVwapUnder, title="Short Entry", message="Short Entry")
