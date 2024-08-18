//@version=5
strategy('yBB Trade', overlay=true, precision=2, default_qty_type=strategy.percent_of_equity, default_qty_value=100, commission_type='percent', commission_value=0.0, pyramiding=1, calc_on_every_tick=true, process_orders_on_close=true)

// ║            CONSTANTS                 ║
Ver = 'V 0.13'

// === INPUT BACKTEST RANGE ===
fromYear = input.int(defval=2010, title='From Year', minval=2010)
thruYear = input.int(defval=2025, title='Thru Year', minval=2010, maxval=2025)
// **** INPUT Section *****
tradeType = input.string(defval = 'Break Up',title='Trade Type',options=['Break Up','MACD Break High','Swing High','DC Up Trend','Demand/Supply','Linear Regression'])
slType = input.string(defval = 'Last low 2',title='Stop Loss Type',options=['Last low 2','Last low 10','ATR'])
trailType = input.string(defval = 'Linear Regression Mid',title='Trailing Stop Type',options=['Below EMA5','Break down','MACD below Signal','Break Low Demand/Supply','Linear Regression Fast','Linear Regression Mid'])

rRatio = input.float(defval=1.5, title="Reward/Risk Ratio",step=0.1,minval=1.0)
rLimit = input.float(defval=5.0, title="% Risk Limited ",step=1,minval=1)  //mean  ( entry - SL )/entry < rLimited
longtermCandles = input.int(defval=200,title='Longterm candles',step=1,minval=50)
adxTrendTicker = input.int(defval=25,title='ADX strong trend threshold',step=1,minval=20)
MacdTrendTicker = input.int(defval=30,title='MACD strong trend ticker',step=1,minval=10)
lostLimit = input.int(defval=1000,title='Lost limitd (Baht) per trade',step=100,minval=100)
dsPeriod = input.int(defval=5,title='Demand Supply Check Period',step=1,minval=1)

// **** FILTER Section *****
financeFilter = input.bool(defval=false, title='Financing filter')
operationATH_Filter = input.bool(defval=false, title='Operation Income ATH filter')
aboveEma200Filter = input.bool(defval=false, title='Trade above EMA longterm')
ema200UpTrendFilter = input.bool(defval=false, title='EMA longterm uptrend filter')
adxStrongTrendFilter = input.bool(defval=true, title='ADX strong trend filter')
atrRunningFilter = input.bool(defval=true, title='ATR running filter')
swingLowFilter = input.bool(defval=false, title='Avoid Trade Price swing low')
swingHighFilter = input.bool(defval=false, title='Only Trade Price swing high')
macdTrendFilter = input.bool(defval=false, title='MACD break last 20 filter')

// === FUNCTION EXAMPLE ===
start = timestamp(fromYear, 1, 1, 00, 00)  // backtest start window
finish = timestamp(thruYear, 1, 1, 24, 59)  // backtest finish window  
window() =>  // create function "within window of time"
    time >= start and time <= finish ? true : false

lower(ref1, ref2) => ref1 < ref2 ? ref1 : ref2
higher(ref1,ref2) => ref1 > ref2 ? ref1 : ref2

getColor(_buySignal, trans) =>
    _buySignal == 1 ? color.new(color.aqua, trans) : _buySignal == 0 ? color.new(color.white, trans) : color.new(color.red, trans)

_color = color.new(color.white,80)

//##################[ Swing High / Low ]#######################################################
var shortSupport = low
var shortResistance = high
var shortDirection = false // swing high/low direction
tmpHigh = ta.highest(high,3) // get high from last 3 candles, potential resistance if break down
tmpLow = ta.lowest(low,3)   // get low from last 3 candles, potential support if break up
if (close<shortSupport or close<ta.lowest(low[1],9)) or (not shortDirection and low<shortSupport)              // new low or break low, identify resistance
    shortResistance := tmpHigh
    shortDirection := false
    shortSupport := low
if (close>shortResistance or close>ta.highest(high[1],9)) or (shortDirection and high>shortResistance)    // new high or break high, identify support
    shortSupport := tmpLow
    shortDirection := true
    shortResistance := high
plot(shortDirection?shortSupport:na,color=color.aqua,style=plot.style_linebr,linewidth=1,title='Swing High',display=display.none)
plot(shortDirection?na:shortResistance,color=color.orange,style=plot.style_linebr,linewidth=1,title='Swing Low',display=display.none)
//##################[ Swing High / Low ]#######################################################

getLowBody(period) => lower(ta.lowest(open,period),ta.lowest(close,period))
getHighBody(period) => higher(ta.highest(open,period),ta.highest(close,period))

//##################[ General Indicators ]#######################################################
[diplus, diminus, adx] = ta.dmi(17, 14)
// [macdLine, signalLine, histLine] = ta.macd(close, 60, 130, 45)
[macdLine, signalLine, histLine] = ta.macd(close, 12, 26, 9)
_atr = ta.atr(10)
_atrShort = ta.ema(_atr,dsPeriod)
var dcLongDirection = false
var dcMidDirection = false
var dcShortDirection = false
if close<ta.lowest(low[1],longtermCandles)
    dcLongDirection := false
if close>ta.highest(high[1],longtermCandles)
    dcLongDirection := true
if close<ta.lowest(low[1],50)
    dcMidDirection := false
if close>ta.highest(high[1],50)
    dcMidDirection := true
if close<ta.lowest(low[1],20)
    dcShortDirection := false
if close>ta.highest(high[1],20)
    dcShortDirection := true

ema5 = ta.ema(close,5)
ema200 = ta.ema(close, longtermCandles)
plot(ema200, title='EMA long term', color=ema200 >= ema200[1] ? color.aqua : color.orange, linewidth=1,display=display.none)
v100 = ta.sma (volume,100)

//################[ Plot H/L Boxes ]#########################################################
var hBox = box.new(bar_index, open, bar_index, close,extend=extend.right,border_width=2,bgcolor=_color,border_color=_color)
var lBox = box.new(bar_index, open, bar_index, close,extend=extend.right,border_width=2,bgcolor=_color,border_color=_color)
var hBox2 = box.new(bar_index, open, bar_index, close,extend=extend.right,border_width=2,bgcolor=_color,border_color=_color)
var lBox2 = box.new(bar_index, open, bar_index, close,extend=extend.right,border_width=2,bgcolor=_color,border_color=_color)
//  **** Update Potential H/L *****
_oh = ta.highest(open,11) // filter open-close or body high
_ch = ta.highest(close,11)
_h = _oh>_ch ? _oh : _ch // return highest body
__h = ta.highest(high,11)

_ol = ta.lowest(open,11) // filter open-close or body low
_cl = ta.lowest(close,11)
_l = _ol<_cl ? _ol : _cl
__l = ta.lowest(low,11)

//##################[ Swing EMA5 H/L ]#######################################################
overEma5 = ta.crossover(close,ema5)
underEma5 = ta.crossunder(close,ema5)
var lowEma5Direction = 0 // -1:UpTrend, 0:Sideway, 1:DownTrend
var highEma5Direction = 0 // -1:UpTrend, 0:Sideway, 1:DownTrend
var pLow = low
var pHigh = high
lBody5 = getLowBody(5)
hBody5 = getHighBody(5)
hlBuff = (pHigh-pLow)*.2  // threshold to identify significant enough for not sideway
if underEma5        // break down
    pHigh := hBody5
    highDiff = (pHigh-pHigh[1])  // diff current high and previous high
    highEma5Direction := highDiff>hlBuff ? -1 : (-highDiff>hlBuff? 1:0 )   // Up, Down, Sideway
else if overEma5    // break up
    pLow := lBody5
    lowDiff = (pLow-pLow[1])    // diff current low and previous low
    lowEma5Direction := lowDiff>hlBuff ? -1 : (-lowDiff>hlBuff? 1:0 )   // Up, Down, Sideway
if close+hlBuff < pLow    // new low
    lowEma5Direction := 1
    if close<pLow[1] and _atrShort>_atrShort[1]  //panic Sell
        highEma5Direction := 1
if close-hlBuff > pHigh    // new high
    highEma5Direction := -1
    if close>pHigh[1] and _atrShort>_atrShort[1] //panic buy
        lowEma5Direction := -1
swingEma5TrendDirection = (lowEma5Direction<=0 and highEma5Direction==-1) ? -1: ((lowEma5Direction==1) and (highEma5Direction>=0)? 1:0)
plot(tradeType=='Swing High'? (underEma5?pHigh:(overEma5?pLow:na)):na,offset=-1,title='swing H/L',style=plot.style_line,color=color.silver,linewidth=1)
plotshape(true,location=location.top,style=shape.circle,title="Swing EMA5 Trend",color=swingEma5TrendDirection<0?color.green:(swingEma5TrendDirection>0?color.red:color.gray))
//##################[ Swing EMA5 H/L ]#######################################################

//#############[ Break supertrend ]#####################
[supertrendFast,supertrendDirectionFast] = ta.supertrend(2,5)
plot(supertrendFast, "Swing Fast direction", color =_color, style=plot.style_linebr,linewidth=1,display=display.none)
var lowDirection = false // false= lower low, true= lower high
var highDirection = false // false= higher low, true= higher high
if ta.crossunder(close,box.get_bottom(lBox))
    lowDirection := false
if ta.crossover(close,box.get_top(hBox))
    highDirection := true
if close>ta.highest(high[1],100) // break up long term range, clear false lower low
    lowDirection := true
if close<ta.lowest(low[1],100)  // break down long term range, clear false higher high
    highDirection := false

swingTrendDirection = lowDirection and highDirection ? -1: ((not lowDirection) and (not highDirection)? 1:0)
// if supertrendDirectionFast < 0 and supertrendDirectionFast[1] > 0// Break Up
if shortDirection==true and shortDirection[1]==false     // Break Up
    gTop = box.get_top(lBox)
    gBottom = box.get_bottom(lBox)
    // check lower low or lower high
    lowDirection := __l > gTop ? true:false
    // Break up, plot low
    line.new(x1=bar_index-10, y1=_l, x2=bar_index+20, y2=_l,color=_color,width=4)
    box.set_lefttop(lBox2, box.get_left(lBox), gTop)
    box.set_rightbottom(lBox2, box.get_right(lBox), gBottom)
    box.set_border_color(lBox2, color= gTop!=gBottom ? na : _color)
    box.set_bgcolor(lBox2,color=_color)

    box.set_lefttop(lBox, bar_index-10, _l)
    box.set_rightbottom(lBox, bar_index-10, __l)
    box.set_border_color(lBox, color= _l!=__l ? na : _color)
    box.set_bgcolor(lBox,color=_color)
else if shortDirection==false and shortDirection[1]==true       // break down
// else if supertrendDirectionFast > 0 and supertrendDirectionFast[1] < 0// break down
    gTop = box.get_top(hBox)
    gBottom = box.get_bottom(hBox)
    // check higher low or higher high
    highDirection := __h < gBottom ? false:true
    // Break down, plot high
    line.new(x1=bar_index-10, y1=_h, x2=bar_index+20, y2=_h,color=_color,width=4)
    box.set_lefttop(hBox2,box.get_left(hBox), gTop)
    box.set_rightbottom(hBox2, box.get_right(hBox), gBottom)
    box.set_border_color(hBox2, color= gTop!=gBottom ? na : _color)
    box.set_bgcolor(hBox2,color=_color)

    box.set_lefttop(hBox,bar_index-10, __h)
    box.set_rightbottom(hBox, bar_index-10, _h)
    box.set_border_color(hBox, color= _h!=__h ? na : _color)
    box.set_bgcolor(hBox,color=_color)

plot(shortDirection!=shortDirection[1] and shortDirection ?_l:na,title='Short Buy',color=_color,style=plot.style_circles,linewidth=1)
plot(shortDirection!=shortDirection[1] and not shortDirection ? _h:na,title='Short Sell',color=_color,style=plot.style_circles,linewidth=1)
plotshape(true,location=location.top,style=shape.square,title="Swing Trend",color=swingTrendDirection<0?color.green:(swingTrendDirection>0?color.red:color.gray),display=display.none)
//###########################################################################
// ***** Financial Section ******
ts = nz(request.financial(syminfo.tickerid, 'TOTAL_SHARES_OUTSTANDING', 'FQ', ignore_invalid_symbol=true))
ff = nz(request.financial(syminfo.tickerid, 'FLOAT_SHARES_OUTSTANDING', 'FY', ignore_invalid_symbol=true))
EPS = nz(request.financial(syminfo.tickerid, 'EARNINGS_PER_SHARE', 'TTM', ignore_invalid_symbol=true))
EPSfq = nz(request.financial(syminfo.tickerid, 'EARNINGS_PER_SHARE', 'FQ', ignore_invalid_symbol=true))
BVPS = nz(request.financial(syminfo.tickerid, 'BOOK_VALUE_PER_SHARE', 'FQ', ignore_invalid_symbol=true))
ROE = nz(request.financial(syminfo.tickerid, 'RETURN_ON_EQUITY', 'FQ', ignore_invalid_symbol=true))
DE = nz(request.financial(syminfo.tickerid, 'DEBT_TO_EQUITY', 'FQ', ignore_invalid_symbol=true))
// GM = nz(request.financial(syminfo.tickerid, 'GROSS_MARGIN', 'FQ', ignore_invalid_symbol=true))
ICR = nz(request.financial(syminfo.tickerid, 'INTERST_COVER', 'FQ', ignore_invalid_symbol=true))
DEVIDEND = nz(request.financial(syminfo.tickerid, 'DIVIDENDS_YIELD', 'FY', ignore_invalid_symbol=true))
FreeCashFlow = nz(request.financial(syminfo.tickerid, 'FREE_CASH_FLOW', 'TTM', ignore_invalid_symbol=true))
CashOperating = nz(request.financial(syminfo.tickerid, 'CASH_F_OPERATING_ACTIVITIES', 'TTM', ignore_invalid_symbol=true))
CashInvesting = nz(request.financial(syminfo.tickerid, 'CASH_F_INVESTING_ACTIVITIES', 'TTM', ignore_invalid_symbol=true))
CashFinancing = nz(request.financial(syminfo.tickerid, 'CASH_F_FINANCING_ACTIVITIES', 'TTM', ignore_invalid_symbol=true))
OperatingIncome = nz(request.financial(syminfo.tickerid, 'OPER_INCOME', 'TTM', ignore_invalid_symbol=true))
var diffOperatingIncome = 0.0  // true is up trend, false is down trend
if OperatingIncome != OperatingIncome[1]
    diffOperatingIncome := (OperatingIncome - OperatingIncome[1])/math.abs(OperatingIncome[1])*100

isFinanceWarning = EPS<0 or ROE<0 or DE>2 or OperatingIncome<0 or diffOperatingIncome<0.1 or (ICR!=0 and ICR<4) or FreeCashFlow<0 or CashOperating<0 or CashOperating < -CashInvesting/2 ? true:false
isFreeCashFlowATH = FreeCashFlow>0 and FreeCashFlow>=ta.highest(FreeCashFlow[1],200) ? true:false
isOperatingIncomeATH = OperatingIncome>0 and OperatingIncome>=ta.highest(OperatingIncome[1],200) ? true:false
isGrowthEPS = EPS>0 and EPS>=ta.highest(EPS[1],200) ? true:false
plotchar(OperatingIncome!=OperatingIncome[1] and diffOperatingIncome>20,textcolor=color.green,location=location.top,text='OP++',title='Operation Income Increase >20%')
plotchar(OperatingIncome!=OperatingIncome[1] and diffOperatingIncome<-20,textcolor=color.red,location=location.top,text='OP--',title='Operation Income Decrease >20%')
//##################################################################################################################
// ****** Print Table *****
var tLog = table.new(position=position.bottom_left, rows=24, columns=2, bgcolor=color.black, border_width=0)
var financeWarning = table.new(position=position.bottom_right, rows=1, columns=1, bgcolor=color.red, border_width=0)
// _avgTradeValue = nz(ta.sma(ohlc4, 100) * v100)
if barstate.islast
    table.cell(tLog, row=0, column=0, text='PE', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=0, column=1, text=str.tostring(close / EPS, '#'), text_color=EPSfq / (EPS / 4) > 1.1 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=1, column=0, text='EPS', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=1, column=1, text=str.tostring(EPS, '#.##'), text_color=isGrowthEPS ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=2, column=0, text='ROE', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=2, column=1, text=str.tostring(ROE, '#'), text_color=ROE > 10 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=3, column=0, text='DE', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=3, column=1, text=str.tostring(DE, '#.#'), text_color=DE < 2 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=4, column=0, text='OP Diff', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=4, column=1, text=str.tostring(diffOperatingIncome,'#')+'%', text_color=diffOperatingIncome>0 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=5, column=0, text='ICR', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=5, column=1, text=str.tostring(ICR, '#.#'), text_color=ICR > 5 ? color.aqua : color.orange, text_size=size.auto)
    _pff = nz(ff / ts * 100)
    table.cell(tLog, row=10, column=0, text='%FF', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=10, column=1, text=str.tostring(_pff, '#'), text_color=_pff < 50 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=11, column=0, text='aVA', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=11, column=1, text=str.tostring(ohlc4*v100, format.volume), text_color=ohlc4* v100 > 10000000 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=12, column=0, text='%YD', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=12, column=1, text=str.tostring(DEVIDEND, "#.#"), text_color=DEVIDEND > 4 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=13, column=0, text='Cash', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=13, column=1, text=str.tostring(FreeCashFlow, format.volume), text_color=FreeCashFlow > 0 ? (isFreeCashFlowATH?color.aqua:color.white) : color.orange, text_size=size.auto)
    table.cell(tLog, row=14, column=0, text='CashOp', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=14, column=1, text=str.tostring(CashOperating, format.volume), text_color=CashOperating > 0 ? (diffOperatingIncome>0?color.aqua:color.white) : color.orange, text_size=size.auto)
    table.cell(tLog, row=15, column=0, text='CashInv', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=15, column=1, text=str.tostring(CashInvesting, format.volume), text_color=CashInvesting+CashOperating > 0 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(tLog, row=16, column=0, text='CashFN', text_color=color.silver, text_size=size.auto)
    table.cell(tLog, row=16, column=1, text=str.tostring(CashFinancing, format.volume), text_color=CashFinancing > 0 ? color.aqua : color.orange, text_size=size.auto)
    table.cell(financeWarning, row=0, column=0, text=isFinanceWarning?'FN WARNING':'GROWTH', text_color=isFinanceWarning ? color.yellow:(isOperatingIncomeATH?color.white:na), text_size=size.auto)

// setROC = request.security('SET:SET', timeframe.period, ta.roc(close, shortPeriod))
// set50ROC = request.security('SET:SET50', timeframe.period, ta.roc(close, shortPeriod))
// maiROC = request.security('SET:MAI', timeframe.period, ta.roc(close, shortPeriod))
// stockROC = ta.roc(close, shortPeriod)
// obv20 = ta.rma(ta.rma(ta.obv, shortPeriod), shortPeriod)
// table.cell(tLog, row=12, column=0, text='ROC', text_color=color.silver, text_size=size.auto)
// table.cell(tLog, row=12, column=1, text=str.tostring(stockROC, '#.#') + '%', text_color=stockROC > 0 ? color.aqua : color.orange, text_size=size.auto)
// table.cell(tLog, row=13, column=0, text='SET', text_color=color.silver, text_size=size.auto)
// table.cell(tLog, row=13, column=1, text=str.tostring(setROC, '#.#') + '%', text_color=setROC > 0 ? color.aqua : color.orange, text_size=size.auto)
// table.cell(tLog, row=14, column=0, text='SET50', text_color=color.silver, text_size=size.auto)
// table.cell(tLog, row=14, column=1, text=str.tostring(set50ROC, '#.#') + '%', text_color=set50ROC > 0 ? color.aqua : color.orange, text_size=size.auto)
// table.cell(tLog, row=15, column=0, text='MAI', text_color=color.silver, text_size=size.auto)
// table.cell(tLog, row=15, column=1, text=str.tostring(maiROC, '#.#') + '%', text_color=maiROC > 0 ? color.aqua : color.orange, text_size=size.auto)

// table.cell(tLog, row=17, column=0, text='OBV', text_color=color.silver, text_size=size.auto)
// table.cell(tLog, row=17, column=1, text=obv20 > obv20[1] ? 'up' : 'dw', text_color=obv20 > obv20[1] ? color.aqua : color.orange, text_size=size.auto)
// table.cell(tLog, row=18, column=0, text='Trend', text_color=color.silver, text_size=size.auto)
// table.cell(tLog, row=18, column=1, text=s200 == 1 ? 'UP' : s200 < 0 ? 'DW' : 'SW', text_color=s200 == 1 ? color.lime : s200 < 0 ? color.red : color.gray, text_size=size.auto)
// table.cell(tLog, row=19, column=0, text=window() ? Ver : 'Expired Contact Developper to Extend', text_color=color.white, text_size=size.small)

//###############################[ Identify potential trading up ]###########################################

//###############################[ Identify Demand Supply Zone ]###########################################
//###### Demand Supply logic , begin railly, long body, break out from sideway=> break DC20, break ATR supertrend*3
var dsLow = low // keep demand/supply low
var dsHigh = high // keep demand/supply high
rc = ta.roc(close,dsPeriod)
[_, upperRC, lowerRC] = ta.bb(rc, 50, 1.9)

// isDemandSupplyZone = ta.tr/ta.tr[1]>2 and (ta.tr/_atr[1] > 3 or supertrendDirectionFast != supertrendDirectionFast[1] or ta.crossunder(rc,lowerRC) or ta.crossover(rc,upperRC) )   // long body,

// break DC20 and find 1st close cross ema5
isDemandZone = ta.crossover(close,ta.highest(high[1],20))
idEma5 = ta.barssince(close<ema5)  // index demand close cross EMA5
idRC = ta.barssince(rc>upperRC)  // index RC cross upper/lower band = strong momentum move

isSupplyZone = ta.crossunder(close,ta.lowest(low[1],20))
isEma5 = ta.barssince(close>ema5)  // index supply close cross EMA5
isRC = ta.barssince(rc<lowerRC)  // index RC cross upper/lower band = strong momentum move

if (isDemandZone and idRC<=idEma5)  // demand zone
    if high[idEma5] != dsHigh and low[idEma5] != dsLow // identify not duplicated DS zone
        // label.new(bar_index,high,text=str.tostring(idEma5),style=label.style_circle,textcolor=color.white,color=na)
        dsLow := low[idEma5]
        dsHigh := high[idEma5]
        box.new(bar_index-idEma5, high[idEma5], bar_index+200, low[idEma5],border_width=0,bgcolor=color.new(color.aqua,80))
else if (isSupplyZone and isRC<=isEma5) // supply zone
    if high[isEma5] != dsHigh and low[isEma5] != dsLow // identify not duplicated DS zone
        // label.new(bar_index,low,text=str.tostring(isEma5),style=label.style_circle,textcolor=color.white,color=na)
        dsLow := low[isEma5]
        dsHigh := high[isEma5]
        box.new(bar_index-isEma5, high[isEma5], bar_index+200, low[isEma5],border_width=0,bgcolor=color.new(color.orange,80))

// if ta.tr/_atr[3] > 2.5 and ta.tr/ta.tr[3]>3
//     box.new(bar_index-3, high[3], bar_index+200, low[3],border_width=0,bgcolor=color.new(color.white,80))



//##################[ Trade at Demand / Supply ]#######################################################
isBreakLowSupport() =>       // for SLPrice
    bool result = false
    for i=0 to longtermCandles
        if close<dsLow[i] and close<open and high<=high[1] and low<low[1] and ( high>dsLow[i] or high[1]>dsLow[i] or high[2]>dsLow[i] or high[3]>dsLow[i] or high[4]>dsLow[i])
            result := true
            break
    result
_isBreakLowSupport = isBreakLowSupport()

isBreakHighResistance() =>       // for SLPrice
    bool result = false
    for i=0 to longtermCandles
        if close>dsHigh[i] and close>open and high>=high[1] and low>low[1] and ( low<dsHigh[i] or low[1]<dsHigh[i] or low[2]<dsHigh[i] or low[3]<dsHigh[i] or low[4]<dsHigh[i])
            result := true
            break
    result
_isBreakHighResistance = isBreakHighResistance()

//##################[ TRADE FILTER ]#######################################################
bcloseShort = ta.linreg(close, 30, 0)
signalShort = ta.sma(bcloseShort, 9)
bcloseMid = ta.linreg(close, 100, 0)
signalMid = ta.sma(bcloseMid, 20)
bcloseLong = ta.linreg(close, 200, 0)
signalLong = ta.sma(bcloseLong, 30)
tFin        = financeFilter? not isFinanceWarning :true
tOp         = operationATH_Filter ? isOperatingIncomeATH : true
tAboveEma   = aboveEma200Filter ? close>ema200 : true
tUpTrendEma = ema200UpTrendFilter ? ema200>ema200[1] : true
tStrongADX  = adxStrongTrendFilter ? adx>adxTrendTicker : true
tATRrunning = atrRunningFilter ? _atrShort>_atrShort[1] : true
tNoSwingL   = swingLowFilter ? swingTrendDirection<=0 : true  // no trade swing low, only sideway or swing high
tOnlySwingH = swingHighFilter ? swingTrendDirection<0 : true  // only trade swing high
macdCrossO  = ta.crossover(macdLine,ta.highest(macdLine[1],MacdTrendTicker*2))  // 20 period confirm break up -> Open Trade
macdCrossU  = ta.crossunder(macdLine,0) or ta.crossunder(macdLine,ta.lowest(macdLine[1],MacdTrendTicker)) // 10 period break down -> close Trade
tMacdBreakH = macdTrendFilter ? macdCrossO : true
tradeFilter = tFin and tOp and tAboveEma and tUpTrendEma and tStrongADX and tATRrunning and tNoSwingL and tOnlySwingH and tMacdBreakH and window()
tDonchain   = dcLongDirection and dcMidDirection

// trailType = input.string(defval = 'Below EMA5',title='Trailing Stop Type',options=['Below EMA5','Break down','MACD belowSignal','Break Low Demand/Supply','Linear Regression Fast','Linear Regression Mid'])
isTrailStop = trailType=='Below EMA5'? underEma5: trailType=='Break down'? not shortDirection: trailType=='MACD below Signal'? macdLine<signalLine: trailType=='Break Low Demand/Supply'? _isBreakLowSupport: trailType=='Linear Regression Fast'?close<signalShort: trailType=='Linear Regression Mid'?close<signalMid :false

normalRisk = close - ta.lowest(low[1],2)
risk = low > high[1] ? close-high[1] : normalRisk  // calculate risk find low from last 2 candles, reduced risk in case a gap

//##################[ 'Linear Regression' TRADING ]#######################################################
tLogic6 = signalMid>signalLong and close>signalShort and close>signalMid and close>signalLong and close>open

var isOpenTradeLR = false // trade pending status one at a time
var entryPriceLR = 0.0
var tpPriceLR = 0.0
var slPriceLR = 0.0
var winLR = 0
var tradeCounterLR = 0
var earnLR = 1.0  // 100% percent
var _lbLR = label.new(bar_index,open,style=label.style_label_down,color=color.new(color.white,80))
var tGainLR = 0.0
var tLostLR = 0.0

//  Open Trade Section
if tradeFilter and not isOpenTradeLR and tLogic6  // short break up trade type
    entryPriceLR := close
    slPriceLR := slType=='Last low 2' ? close - risk : slType=='Last low 10' ? __l : close-2*_atr
    tpPriceLR := close + (close-slPriceLR)*rRatio
    tGainLR := (tpPriceLR-entryPriceLR)/entryPriceLR
    tLostLR := (entryPriceLR-slPriceLR)/entryPriceLR
    isOpenTradeLR := true     // open trade
    if tradeType=='Linear Regression'
        _lbLR := label.new(bar_index,tpPriceLR,color=_color,textcolor=color.yellow,text="WR "+str.tostring(winLR/tradeCounterLR,"#%")+"\nTP "+str.tostring(tpPriceLR,format.mintick)+"\nSL "+str.tostring(slPriceLR,format.mintick)+"\nBuy "+str.tostring(math.round(lostLimit/(entryPriceLR-slPriceLR)/100)*100,format.volume),tooltip='Lost limit '+str.tostring(lostLimit,'# baht'))
    tradeCounterLR := tradeCounterLR + 1
//  Close Trade Section
if isOpenTradeLR
    if high>=tpPriceLR               // exit trade hit TP target price
        isOpenTradeLR := false    // close or exit trade
        winLR := winLR+1
        gain = tLostLR<rLimit/100.0? tGainLR :rLimit*rRatio/100  // rLimit is not in percent yet, .98 is comission
        earnLR := earnLR*(1+gain) // percent gain from risk limit * RR ratio
        tooltipText = str.tostring(gain,'#.##%')+'\nWR '+str.tostring(winLR/tradeCounterLR,'#%\nW')+str.tostring(winLR,'#')+'/'+str.tostring(tradeCounterLR,'#')+'\nE '+str.tostring(earnLR-1,'#%')
        label.set_tooltip(_lbLR,tooltip=label.get_text(_lbLR)+'\n'+tooltipText)
        label.set_text(_lbLR,text=str.tostring(gain,'#.#%'))
        label.set_textcolor(_lbLR,textcolor=color.aqua)
    else if close<slPriceLR  // close<sLPrice = rLimit or Risk limit, ie: 5%
        isOpenTradeLR := false    // close or exit trade
        lost = tLostLR<rLimit/100.0? -tLostLR : -rLimit/100
        earnLR := earnLR*(1+lost) // reduce from risk limit
        tooltipText = str.tostring(lost,'#.##%')+'\nWR '+str.tostring(winLR/tradeCounterLR,'#%\nW')+str.tostring(winLR,'#')+'/'+str.tostring(tradeCounterLR,'#')+'\nE '+str.tostring(earnLR-1,'#%')
        label.set_tooltip(_lbLR,tooltip=label.get_text(_lbLR)+'\n'+tooltipText)
        label.set_text(_lbLR,text=str.tostring(lost,'#.#%'))
        label.set_textcolor(_lbLR,textcolor=color.orange)

entryLineLR = plot(tradeType=='Linear Regression' and isOpenTradeLR? entryPriceLR:na,color=color.yellow,title='Linear Regression Entry Price',style=plot.style_linebr)
slLineLR = plot(tradeType=='Linear Regression' and isOpenTradeLR? slPriceLR:na,color=color.red,title='Linear Regression Stop Loss Price',style=plot.style_linebr)
tpLineLR = plot(tradeType=='Linear Regression' and isOpenTradeLR? tpPriceLR:na,color=color.aqua,title='Linear Regression Target Price',style=plot.style_linebr)
fill(entryLineLR,tpLineLR,color=_color)
fill(entryLineLR,slLineLR,color=_color)
plot(signalShort,color=color.gray,title='Linear Regression Short',display=display.none)
plot(signalMid,color=color.white,title='Linear Regression Mid')
plot(signalLong,color=color.purple,title='Linear Regression Long',display=display.none)
//##################[ 'Linear Regression' TRADING ]#######################################################

//##################[ 'Demand/Supply' TRADING ]#######################################################
tLogic5 = isBreakHighResistance() and close>open    // Break Demand/Supply
var isOpenTradeDS = false // trade pending status one at a time
var entryPriceDS = 0.0
var tpPriceDS = 0.0
var slPriceDS = 0.0
var winDS = 0
var tradeCounterDS = 0
var earnDS = 1.0  // 100% percent
var _lbDS = label.new(bar_index,open,style=label.style_label_down,color=color.new(color.white,80))
var tGainDS = 0.0
var tLostDS = 0.0

//  Open Trade Section
if tradeFilter and not isOpenTradeDS and tLogic5  // rebound from support DS line
    entryPriceDS := close
    slPriceDS := slType=='Last low 2' ? close - risk : slType=='Last low 10' ? __l : close-2*_atr
    tpPriceDS := close + (close-slPriceDS)*rRatio
    tGainDS := (tpPriceDS-entryPriceDS)/entryPriceDS
    tLostDS := (entryPriceDS-slPriceDS)/entryPriceDS
    isOpenTradeDS := true     // open trade
    if tradeType=='Demand/Supply'
        _lbDS := label.new(bar_index,tpPriceDS,color=_color,textcolor=color.yellow,text="WR "+str.tostring(winDS/tradeCounterDS,"#%")+"\nTP "+str.tostring(tpPriceDS,format.mintick)+"\nSL "+str.tostring(slPriceDS,format.mintick)+"\nBuy "+str.tostring(math.round(lostLimit/(entryPriceDS-slPriceDS)/100)*100,format.volume),tooltip='Lost limit '+str.tostring(lostLimit,'# baht'))
    tradeCounterDS := tradeCounterDS + 1
//  Close Trade Section
if isOpenTradeDS
    if high>=tpPriceDS               // exit trade hit TP target price
        isOpenTradeDS := false    // close or exit trade
        winDS := winDS+1
        gain = tGainDS
        earnDS := earnDS*(1+gain) // percent gain from risk limit * RR ratio
        tooltipText = 'W '+str.tostring(winDS,'#')+'/'+str.tostring(tradeCounterDS,'#')+'\nE '+str.tostring(earnDS-1,'#%')
        label.set_tooltip(_lbDS,tooltip=label.get_text(_lbDS)+'\n'+tooltipText)
        label.set_text(_lbDS,text=str.tostring(gain,'#.#%'))
        label.set_textcolor(_lbDS,textcolor=color.aqua)
    else if close<slPriceDS  // close<sLPrice = rLimit or Risk limit, ie: 5%
        isOpenTradeDS := false    // close or exit trade
        lost = -tLostDS
        earnDS := earnDS*(1+lost) // reduce from risk limit
        tooltipText = 'W '+str.tostring(winDS,'#')+'/'+str.tostring(tradeCounterDS,'#')+'\nE '+str.tostring(earnDS-1,'#%')
        label.set_tooltip(_lbDS,tooltip=label.get_text(_lbDS)+'\n'+tooltipText)
        label.set_text(_lbDS,text=str.tostring(lost,'#.#%'))
        label.set_textcolor(_lbDS,textcolor=color.orange)
    else if _isBreakLowSupport     // stop lost break low support
        isOpenTradeDS := false    // close or exit trade
        gain = (close-entryPriceDS)/entryPriceDS  // rLimit is not in percent yet, .98 is comission
        if gain>0
            winDS := winDS+1
        earnDS := earnDS*(1+gain) // percent gain from risk limit * RR ratio
        tooltipText = str.tostring(gain,'#.#%')+'\nW '+str.tostring(winDS,'#')+'/'+str.tostring(tradeCounterDS,'#')+'\nE '+str.tostring(earnDS-1,'#%')
        label.set_tooltip(_lbDS,tooltip=label.get_text(_lbDS)+'\n'+tooltipText)
        label.set_text(_lbDS,text=str.tostring(gain,'#.#%'))
        label.set_textcolor(_lbDS,textcolor=gain>0?color.aqua:color.orange)

entryLineDS = plot(tradeType=='Demand/Supply' and isOpenTradeDS? entryPriceDS:na,color=color.yellow,title='DS Entry Price',style=plot.style_linebr)
slLineDS = plot(tradeType=='Demand/Supply' and isOpenTradeDS? slPriceDS:na,color=color.red,title='DS Stop Loss Price',style=plot.style_linebr)
tpLineDS = plot(tradeType=='Demand/Supply' and isOpenTradeDS? tpPriceDS:na,color=color.aqua,title='DS Target Price',style=plot.style_linebr)
fill(entryLineDS,tpLineDS,color=_color)
fill(entryLineDS,slLineDS,color=_color)
plot(tradeType=='Demand/Supply' and isBreakLowSupport()?high:na,title='Break Low Demand/Supply',color=color.orange,style=plot.style_circles,linewidth=2)
plot(tradeType=='Demand/Supply' and isBreakHighResistance()?low:na,title='Break High Demand/Supply',color=color.aqua,style=plot.style_circles,linewidth=2)
//##################[ 'Demand/Supply' TRADING ]#######################################################

//##################[ 'DC Up Trend' TRADING ]#######################################################
tLogic4 = (dcLongDirection or dcShortDirection) and dcMidDirection and shortDirection and shortDirection!=shortDirection[1] and close>open // Donchain up trend Long and Mid, then short swing break up
var isOpenTradeDC = false // trade pending status one at a time
var entryPriceDC = 0.0
var tpPriceDC = 0.0
var slPriceDC = 0.0
var winDC = 0
var tradeCounterDC = 0
var earnDC = 1.0  // 100% percent
var _lbDC = label.new(bar_index,open,style=label.style_label_down,color=color.new(color.white,80))
var tGainDC = 0.0
var tLostDC = 0.0

//  Open Trade Section
if tradeFilter and not isOpenTradeDC and tLogic4  // open trade DC uptrend and swing short break up
    entryPriceDC := close
    slPriceDC := slType=='Last low 2' ? close - risk : slType=='Last low 10' ? __l : close-2*_atr
    tpPriceDC := close + (close-slPriceDC)*rRatio
    tGainDC := (tpPriceDC-entryPriceDC)/entryPriceDC
    tLostDC := (entryPriceDC-slPriceDC)/entryPriceDC
    isOpenTradeDC := true     // open trade
    if tradeType=='DC Up Trend'
        _lbDC := label.new(bar_index,tpPriceDC,color=_color,textcolor=color.yellow,text="WR "+str.tostring(winDC/tradeCounterDC,"#%")+"\nSL "+str.tostring(slPriceDC,format.mintick)+"\nBuy "+str.tostring(math.round(lostLimit/(entryPriceDC-slPriceDC)/100)*100,format.volume),tooltip='Lost limit '+str.tostring(lostLimit,'# baht'))
    tradeCounterDC := tradeCounterDC + 1
//  Close Trade Section
if isOpenTradeDC
    if not dcShortDirection or (close>entryPriceDC and isTrailStop)          // exit trade in DC20 break down
        isOpenTradeDC := false    // close or exit trade
        gain = (close-entryPriceDC)/entryPriceDC
        if gain>0
            winDC := winDC+1
        earnDC := earnDC*(1+gain) // percent gain from risk limit * RR ratio
        tooltipText = str.tostring(gain,'#.#%')+'\nW '+str.tostring(winDC,'#')+'/'+str.tostring(tradeCounterDC,'#')+'\nE '+str.tostring(earnDC-1,'#%')
        if tradeType=='DC Up Trend'
            label.set_xy(_lbDC,bar_index,high+_atr*2)
            label.set_tooltip(_lbDC,tooltip=label.get_text(_lbDC)+'\n'+tooltipText)
            label.set_text(_lbDC,text=str.tostring(gain,'#.#%'))
            label.set_textcolor(_lbDC,textcolor=gain>0?color.aqua:color.orange)
    else if close<slPriceDC  // close<sLPrice = rLimit or Risk limit, ie: 5%
        isOpenTradeDC := false    // close or exit trade
        lost = -tLostDC
        earnDC := earnDC*(1+lost) // reduce from risk limit
        tooltipText = 'W '+str.tostring(winDC,'#')+'/'+str.tostring(tradeCounterDC,'#')+'\nE '+str.tostring(earnDC-1,'#%')
        label.set_tooltip(_lbDC,tooltip=label.get_text(_lbDC)+'\n'+tooltipText)
        label.set_text(_lbDC,text=str.tostring(lost,'#.#%'))
        label.set_textcolor(_lbDC,textcolor=color.orange)

entryLineDC = plot(tradeType=='DC Up Trend' and isOpenTradeDC? entryPriceDC:na,color=color.yellow,title='Entry Price Donchain Trading',style=plot.style_linebr)
slLineDC = plot(tradeType=='DC Up Trend' and isOpenTradeDC? slPriceDC:na,color=color.red,title='Stop Loss Price Donchain Trading',style=plot.style_linebr)
tpLineDC = plot(tradeType=='DC Up Trend' and isOpenTradeDC? tpPriceDC:na,color=color.aqua,title='Min Target Price Donchain Trading ',style=plot.style_linebr)
fill(entryLineDC,tpLineDC,color=_color)
fill(entryLineDC,slLineDC,color=_color)
//##################[ 'DC Up Trend' TRADING ]#######################################################

//##################[ 'Swing High' TRADING ]#######################################################
// tLogic1 = shortDirection and high>high[1] and low>low[1] and close>open  // short break up trade
// tLogic3 = swingTrendDirection<0 and close>open  // swinng high to open trade only, change to swingEma5TrendDirection
tLogic3 = swingEma5TrendDirection<0 and high>high[1] and low>low[1] and close>open  // swinng high to open trade only, change to swingEma5TrendDirection
var isOpenTradeSwing = false // trade pending status one at a time
var entryPriceSwing = 0.0
var tpPriceSwing = 0.0
var slPriceSwing = 0.0
var winSwing = 0
var tradeCounterSwing = 0
var earnSwing = 1.0  // 100% percent
var _lbSwing = label.new(bar_index,open,style=label.style_label_down,color=color.new(color.white,80))
var tGainSwing = 0.0
var tLostSwing = 0.0

//  Open Trade Section
if tradeFilter and not isOpenTradeSwing and tLogic3  // open trade swing high
    entryPriceSwing := close
    slPriceSwing := slType=='Last low 2' ? close - risk : slType=='Last low 10' ? __l : close-2*_atr
    tpPriceSwing := close + (close-slPriceSwing)*rRatio
    tGainSwing := (tpPriceSwing-entryPriceSwing)/entryPriceSwing
    tLostSwing := (entryPriceSwing-slPriceSwing)/entryPriceSwing
    isOpenTradeSwing := true     // open trade
    if tradeType=='Swing High'
        _lbSwing := label.new(bar_index,tpPriceSwing,color=_color,textcolor=color.yellow,text="WR "+str.tostring(winSwing/tradeCounterSwing,"#%")+"\nSL "+str.tostring(slPriceSwing,format.mintick)+"\nBuy "+str.tostring(math.round(lostLimit/(entryPriceSwing-slPriceSwing)/100)*100,format.volume),tooltip='Lost limit '+str.tostring(lostLimit,'# baht'))
    tradeCounterSwing := tradeCounterSwing + 1
//  Close Trade Section
if isOpenTradeSwing
    // if high>=tpPriceSwing               // exit trade hit TP target price
    //     isOpenTradeSwing := false    // close or exit trade
    //     winSwing := winSwing+1
    //     gain = tLostSwing<rLimit/100.0? tGainSwing : rLimit*rRatio/100.0  // rLimit is not in percent yet
    //     earnSwing := earnSwing*(1+gain) // percent gain from risk limit * RR ratio
    //     tooltipText = str.tostring(gain,'#.##%')+'\nWR '+str.tostring(winSwing/tradeCounterSwing,'#%\nW')+str.tostring(winSwing,'#')+'/'+str.tostring(tradeCounterSwing,'#')+'\nE '+str.tostring(earnSwing-1,'#%')
    //     // tooltipText = 'WR '+str.tostring(win/tradeCounter,'#%\nW')+str.tostring(win,'#')+'/'+str.tostring(tradeCounter,'#')+'\nE '+str.tostring(earn-1,'#%')
    //     label.set_tooltip(_lbSwing,tooltip=label.get_text(_lbSwing)+'\n'+tooltipText)
    //     label.set_text(_lbSwing,text=str.tostring(gain,'#.#%'))
    //     label.set_textcolor(_lbSwing,textcolor=color.aqua)
    if swingEma5TrendDirection>0 or (close>entryPriceSwing and isTrailStop)               // exit trade down trend price Action
        isOpenTradeSwing := false    // close or exit trade
        gain = (close-entryPriceSwing)/entryPriceSwing
        if gain>0
            winSwing := winSwing+1
        earnSwing := earnSwing*(1+gain) // percent gain from risk limit * RR ratio
        tooltipText = str.tostring(gain,'#.#%')+'\nW '+str.tostring(winSwing,'#')+'/'+str.tostring(tradeCounterSwing,'#')+'\nE '+str.tostring(earnSwing-1,'#%')
        if tradeType=='Swing High'
            label.set_xy(_lbSwing,bar_index,high+_atr*2)
            label.set_tooltip(_lbSwing,tooltip=label.get_text(_lbSwing)+'\n'+tooltipText)
            label.set_text(_lbSwing,text=str.tostring(gain,'#.#%'))
            label.set_textcolor(_lbSwing,textcolor=gain>0?color.aqua:color.orange)
    else if close<slPriceSwing  // close<sLPrice = rLimit or Risk limit, ie: 5%
        isOpenTradeSwing := false    // close or exit trade
        lost = -tLostSwing
        earnSwing := earnSwing*(1+lost) // reduce from risk limit
        tooltipText = 'W '+str.tostring(winSwing,'#')+'/'+str.tostring(tradeCounterSwing,'#')+'\nE '+str.tostring(earnSwing-1,'#%')
        label.set_tooltip(_lbSwing,tooltip=label.get_text(_lbSwing)+'\n'+tooltipText)
        label.set_text(_lbSwing,text=str.tostring(lost,'#.#%'))
        label.set_textcolor(_lbSwing,textcolor=color.orange)

entryLineSwing = plot(tradeType=='Swing High' and isOpenTradeSwing? entryPriceSwing:na,color=color.yellow,title='Entry Price Swing Trading',style=plot.style_linebr)
slLineSwing = plot(tradeType=='Swing High' and isOpenTradeSwing? slPriceSwing:na,color=color.red,title='Stop Loss Price Swing Trading',style=plot.style_linebr)
tpLineSwing = plot(tradeType=='Swing High' and isOpenTradeSwing? tpPriceSwing:na,color=color.aqua,title='Min Target Price Swing Trading ',style=plot.style_linebr)
fill(entryLineSwing,tpLineSwing,color=_color)
fill(entryLineSwing,slLineSwing,color=_color)
//##################[ 'Swing High' TRADING ]#######################################################

//##################[ 'MACD Break High' TRADING ]#######################################################
tLogic2 = macdCrossO and close>open  // macd break period high
var isOpenTradeMacd = false // trade pending status one at a time
var entryPriceMacd = 0.0
var tpPriceMacd = 0.0
var slPriceMacd = 0.0
var winMacd = 0
var tradeCounterMacd = 0
var earnMacd = 1.0  // 100% percent
var _lbMacd = label.new(bar_index,open,style=label.style_label_down,color=color.new(color.white,80))
var tGainMacd = 0.0
var tLostMacd = 0.0

//  Open Trade Section
if tradeFilter and not isOpenTradeMacd and tLogic2  // macd break period high
    entryPriceMacd := close
    slPriceMacd := slType=='Last low 2' ? close - risk : slType=='Last low 10' ? __l : close-2*_atr
    tpPriceMacd := close + (close-slPriceMacd)*rRatio
    tGainMacd := (tpPriceMacd-entryPriceMacd)/entryPriceMacd
    tLostMacd := (entryPriceMacd-slPriceMacd)/entryPriceMacd
    isOpenTradeMacd := true     // open trade
    if tradeType=='MACD Break High'
        _lbMacd := label.new(bar_index,tpPriceMacd,color=_color,textcolor=color.yellow,text="WR "+str.tostring(winMacd/tradeCounterMacd,"#%")+"\nSL "+str.tostring(slPriceMacd,format.mintick)+"\nBuy "+str.tostring(math.round(lostLimit/(entryPriceMacd-slPriceMacd)/100)*100,format.volume),tooltip='Lost limit '+str.tostring(lostLimit,'# baht'))
    tradeCounterMacd := tradeCounterMacd + 1
//  Close Trade Section
if isOpenTradeMacd
    if macdCrossU or (close>entryPriceMacd and isTrailStop)              // exit trade macdline break low 10 period
        isOpenTradeMacd := false    // close or exit trade
        gain = (close-entryPriceMacd)/entryPriceMacd
        if gain>0
            winMacd := winMacd+1
        earnMacd := earnMacd*(1+gain) // percent gain from risk limit * RR ratio
        tooltipText = str.tostring(gain,'#.#%')+'\nW '+str.tostring(winMacd,'#')+'/'+str.tostring(tradeCounterMacd,'#')+'\nE '+str.tostring(earnMacd-1,'#%')
        if tradeType=='MACD Break High'
            label.set_xy(_lbMacd,bar_index,high+_atr*2)
            label.set_tooltip(_lbMacd,tooltip=label.get_text(_lbMacd)+'\n'+tooltipText)
            label.set_text(_lbMacd,text=str.tostring(gain,'#.#%'))
            label.set_textcolor(_lbMacd,textcolor=gain>0?color.aqua:color.orange)
    else if close<slPriceMacd  // close<sLPrice = rLimit or Risk limit, ie: 5%
        isOpenTradeMacd := false    // close or exit trade
        lost = -tLostMacd
        earnMacd := earnMacd*(1+lost) // reduce from risk limit
        tooltipText = 'W '+str.tostring(winMacd,'#')+'/'+str.tostring(tradeCounterMacd,'#')+'\nE '+str.tostring(earnMacd-1,'#%')
        label.set_tooltip(_lbMacd,tooltip=label.get_text(_lbMacd)+'\n'+tooltipText)
        label.set_text(_lbMacd,text=str.tostring(lost,'#.#%'))
        label.set_textcolor(_lbMacd,textcolor=color.orange)

entryLineMacd = plot(tradeType=='MACD Break High' and isOpenTradeMacd? entryPriceMacd:na,color=color.yellow,title='Entry Price MACD Trading',style=plot.style_linebr)
slLineMacd = plot(tradeType=='MACD Break High' and isOpenTradeMacd? slPriceMacd:na,color=color.red,title='Stop Loss Price MACD Trading',style=plot.style_linebr)
tpLineMacd = plot(tradeType=='MACD Break High' and isOpenTradeMacd? tpPriceMacd:na,color=color.aqua,title='Min Target Price MACD Trading ',style=plot.style_linebr)
fill(entryLineMacd,tpLineMacd,color=_color)
fill(entryLineMacd,slLineMacd,color=_color)
//##################[ 'MACD Break High' TRADING ]#######################################################

//##################[ 'Break Up' TRADING ]#######################################################
tLogic1 = shortDirection and high>high[1] and low>low[1] and close>open  // short break up trade
var isOpenTrade = false // trade pending status one at a time
var entryPrice = 0.0
var tpPrice = 0.0
var slPrice = 0.0
var win = 0
var tradeCounter = 0
var earn = 1.0  // 100% percent
var _lb = label.new(bar_index,open,style=label.style_label_down,color=color.new(color.white,80))
var tGain = 0.0
var tLost = 0.0
//  Open Trade Section
if tradeFilter and not isOpenTrade and tLogic1  // short break up trade type
    entryPrice := close
    slPrice := slType=='Last low 2' ? close - risk : slType=='Last low 10' ? __l : close-2*_atr
    tpPrice := close + (close-slPrice)*rRatio
    tGain := (tpPrice-entryPrice)/entryPrice
    tLost := (entryPrice-slPrice)/entryPrice
    isOpenTrade := true     // open trade
    if tradeType=='Break Up'
        _lb := label.new(bar_index,tpPrice,color=_color,textcolor=color.yellow,text="WR "+str.tostring(win/tradeCounter,"#%")+"\nTP "+str.tostring(tpPrice,format.mintick)+"\nSL "+str.tostring(slPrice,format.mintick)+"\nBuy "+str.tostring(math.round(lostLimit/(entryPrice-slPrice)/100)*100,format.volume),tooltip='Lost limit '+str.tostring(lostLimit,'# baht'))
    tradeCounter := tradeCounter + 1
//  Close Trade Section
if isOpenTrade
    if high>=tpPrice               // exit trade hit TP target price
        isOpenTrade := false    // close or exit trade
        win := win+1
        gain = tLost<rLimit/100.0? tGain : rLimit*rRatio/100.0  // rLimit is not in percent yet
        earn := earn*(1+gain) // percent gain from risk limit * RR ratio
        tooltipText = str.tostring(gain,'#.##%')+'\nWR '+str.tostring(win/tradeCounter,'#%\nW')+str.tostring(win,'#')+'/'+str.tostring(tradeCounter,'#')+'\nE '+str.tostring(earn-1,'#%')
        // tooltipText = 'WR '+str.tostring(win/tradeCounter,'#%\nW')+str.tostring(win,'#')+'/'+str.tostring(tradeCounter,'#')+'\nE '+str.tostring(earn-1,'#%')
        label.set_tooltip(_lb,tooltip=label.get_text(_lb)+'\n'+tooltipText)
        label.set_text(_lb,text=str.tostring(gain,'#.#%'))
        label.set_textcolor(_lb,textcolor=color.aqua)
    else if close<slPrice  // close<sLPrice = rLimit or Risk limit, ie: 5%
        isOpenTrade := false    // close or exit trade
        lost = tLost<rLimit/100.0? -tLost : -rLimit/100.0
        earn := earn*(1+lost) // reduce from risk limit
        tooltipText = str.tostring(lost,'#.##%')+'\nWR '+str.tostring(win/tradeCounter,'#%\nW')+str.tostring(win,'#')+'/'+str.tostring(tradeCounter,'#')+'\nE '+str.tostring(earn-1,'#%')
        // tooltipText = 'WR '+str.tostring(win/tradeCounter,'#%\nW')+str.tostring(win,'#')+'/'+str.tostring(tradeCounter,'#')+'\nE '+str.tostring(earn-1,'#%')
        label.set_tooltip(_lb,tooltip=label.get_text(_lb)+'\n'+tooltipText)
        label.set_text(_lb,text=str.tostring(lost,'#.#%'))
        label.set_textcolor(_lb,textcolor=color.orange)

entryLine = plot(tradeType=='Break Up' and isOpenTrade? entryPrice:na,color=color.yellow,title='Entry Price',style=plot.style_linebr)
slLine = plot(tradeType=='Break Up' and isOpenTrade? slPrice:na,color=color.red,title='Stop Loss Price',style=plot.style_linebr)
tpLine = plot(tradeType=='Break Up' and isOpenTrade? tpPrice:na,color=color.aqua,title='Target Price',style=plot.style_linebr)
fill(entryLine,tpLine,color=_color)
fill(entryLine,slLine,color=_color)
//##################[ 'Break Up' TRADING ]#######################################################
    

//##################[ TRADING TABLE ]#######################################################
if barstate.islast
    table.cell(tLog, row=17, column=0, text='LI '+str.tostring(winLR,'#/')+str.tostring(tradeCounterLR,'#'), text_color=tradeType=='Linear Regression'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=17, column=1, text=str.tostring(winLR/tradeCounterLR,'w#% ')+str.tostring(earnLR-1,'#%'), text_color=tradeType=='Linear Regression'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=18, column=0, text='BR '+str.tostring(win,'#/')+str.tostring(tradeCounter,'#'), text_color=tradeType=='Break Up'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=18, column=1, text=str.tostring(win/tradeCounter,'w#% ')+str.tostring(earn-1,'#%'), text_color=tradeType=='Break Up'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=19, column=0, text='MC '+str.tostring(winMacd,'#/')+str.tostring(tradeCounterMacd,'#'), text_color=tradeType=='MACD Break High'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=19, column=1, text=str.tostring(winMacd/tradeCounterMacd,'w#% ')+str.tostring(earnMacd-1,'#%'), text_color=tradeType=='MACD Break High'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=20, column=0, text='SW '+str.tostring(winSwing,'#/')+str.tostring(tradeCounterSwing,'#'), text_color=tradeType=='Swing High'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=20, column=1, text=str.tostring(winSwing/tradeCounterSwing,'w#% ')+str.tostring(earnSwing-1,'#%'), text_color=tradeType=='Swing High'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=21, column=0, text='DC '+str.tostring(winDC,'#/')+str.tostring(tradeCounterDC,'#'), text_color=tradeType=='DC Up Trend'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=21, column=1, text=str.tostring(winDC/tradeCounterDC,'w#% ')+str.tostring(earnDC-1,'#%'), text_color=tradeType=='DC Up Trend'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=22, column=0, text='DS '+str.tostring(winDS,'#/')+str.tostring(tradeCounterDS,'#'), text_color=tradeType=='Demand/Supply'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=22, column=1, text=str.tostring(winDS/tradeCounterDS,'w#% ')+str.tostring(earnDS-1,'#%'), text_color=tradeType=='Demand/Supply'?color.yellow:color.white, text_size=size.auto)
    table.cell(tLog, row=23, column=0, text='Ver ', text_color=color.gray, text_size=size.auto)
    table.cell(tLog, row=23, column=1, text=Ver, text_color=color.gray, text_size=size.auto)
//##################[ TRADING TABLE ]#######################################################


// var table ratings = table.new(position.top_right, 8, 2, frame_color = #000000)
// if barstate.islastconfirmedhistory
//     //@variable The time value one year from the date of the last analyst recommendations.
//     int YTD = syminfo.target_price_date + timeframe.in_seconds("12M") * 1000
//     // Add header cells.
//     table.cell(ratings, 0, 0, "Start Date", bgcolor = color.gray, text_color = #000000)
//     table.cell(ratings, 1, 0, "End Date", bgcolor = color.gray, text_color = #000000)
//     table.cell(ratings, 2, 0, "Buy", bgcolor = color.teal, text_color = #000000)
//     table.cell(ratings, 3, 0, "Strong Buy", bgcolor = color.lime, text_color = #000000)
//     table.cell(ratings, 4, 0, "Sell", bgcolor = color.maroon, text_color = #000000)
//     table.cell(ratings, 5, 0, "Strong Sell", bgcolor = color.red, text_color = #000000)
//     table.cell(ratings, 6, 0, "Hold", bgcolor = color.orange, text_color = #000000)

//     table.cell(ratings, 7, 0, "Correlation", bgcolor = color.silver, text_color = #000000)
//     // Recommendation strings
//     string startDate         = str.format_time(syminfo.recommendations_date, "yyyy-MM-dd")
//     string endDate           = str.format_time(YTD, "yyyy-MM-dd")
//     string buyRatings        = str.tostring(syminfo.recommendations_buy)
//     string strongBuyRatings  = str.tostring(syminfo.recommendations_buy_strong)
//     string sellRatings       = str.tostring(syminfo.recommendations_sell)
//     string strongSellRatings = str.tostring(syminfo.recommendations_sell_strong)
//     string holdRatings       = str.tostring(syminfo.recommendations_hold)
//     string totalRatings      = str.tostring(syminfo.recommendations_total)
//     // Add value cells
//     table.cell(ratings, 0, 1, startDate, bgcolor = color.gray, text_color = #000000)
//     table.cell(ratings, 1, 1, endDate, bgcolor = color.gray, text_color = #000000)
//     table.cell(ratings, 2, 1, buyRatings, bgcolor = color.teal, text_color = #000000)
//     table.cell(ratings, 3, 1, strongBuyRatings, bgcolor = color.lime, text_color = #000000)
//     table.cell(ratings, 4, 1, sellRatings, bgcolor = color.maroon, text_color = #000000)

//     table.cell(ratings, 6, 1, str.tostring(lOffset), bgcolor = color.maroon, text_color = #000000)
//     table.cell(ratings, 7, 1, str.tostring(hOffset), bgcolor = color.maroon, text_color = #000000)
