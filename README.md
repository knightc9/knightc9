_SECTION_BEGIN("Price");


SetChartOptions(0,chartShowArrows|chartShowDates);
_N(Title = StrFormat("{{NAME}} - Open %g, Hi %g, Lo %g, Close %g (%.1f%%) {{VALUES}}", O, H, L, C, 
SelectedValue( ROC( C, 1 ) ) )) 
;
Plot( C, "Close", ParamColor("Color", colorDefault ), styleNoTitle | ParamStyle("Style") | GetPriceStyle() );

_SECTION_END();







_SECTION_BEGIN("MA20");
//Plot( MA( Close, 20), "MA-20", colorRed );
Plot( MA( Close, 50), "MA-50", colorBlue );
GraphLabelDecimals = 2;
_SECTION_END();

_SECTION_BEGIN("VOLUME AT PRICE");
PlotVAPOverlay( Param("lines",260,10,1000,1), Param("width",100,1,200,1), ParamColor("color", ColorRGB(214,222,224)), Param("style",2,0,7,1)| 4*ParamToggle("Z-order", "On top|Behind", 1 ) );
_SECTION_END();

_SECTION_BEGIN("VOLUME");
HPercent = Param("Percentage of pane height", 30, 1, 100, 1);
MAV = MA(V, 21);
Vmax =  HighestVisibleValue(V)*100/HPercent;
Vmin = LowestVisibleValue(V);
Plot(MAV, "MAV", colorBlueGrey, styleLine|styleOwnScale, 0, Vmax, 0, 0, 1);
Plot(V, "Volume", IIf(Close >= Open,colorGreen,colorRed), styleHistogram|styleOwnScale, 0, Vmax, 0, 0,2);
_SECTION_END();


_SECTION_END();

_SECTION_BEGIN("Long R/S");

farback = Param("How Far back to go", 100, 50, 5000, 10);
nBars = Param("Number of bars", 12, 5, 40);
aHPivs = H - H;
aLPivs = L - L;
aHPivHighs = H - H;
aLPivLows = L - L;
aHPivIdxs = H - H;
aLPivIdxs = L - L;
nHPivs = 0;
nLPivs = 0;
lastHPIdx = 0;
lastLPIdx = 0;
lastHPH = 0;
lastLPL = 0;
curPivBarIdx = 0;
aHHVBars = HHVBars(H, nBars);
aLLVBars = LLVBars(L, nBars);
aHHV = HHV(H, nBars);
aLLV = LLV(L, nBars);
aVisBars = Status("barvisible");
nLastVisBar = SelectedValue(Highest(IIf(aVisBars, BarIndex(), 0)));
_TRACE("Last visible bar: " + nLastVisBar);
curBar = (BarCount - 1);
curTrend = "";
if (aLLVBars[curBar] < aHHVBars[curBar])
{
    curTrend = "D";
}

else
{
    curTrend = "U";
}

for (i = 0; i < farback; i++)
{
    curBar = (BarCount - 1) - i;
    if (aLLVBars[curBar] < aHHVBars[curBar])
    {
        if (curTrend == "U")
        {
            curTrend = "D";
            curPivBarIdx = curBar - aLLVBars[curBar];
            aLPivs[curPivBarIdx] = 1;
            aLPivLows[nLPivs] = L[curPivBarIdx];
            aLPivIdxs[nLPivs] = curPivBarIdx;
            nLPivs++;
        }
    }
    else
    {
        if (curTrend == "D")
        {
            curTrend = "U";
            curPivBarIdx = curBar - aHHVBars[curBar];
            aHPivs[curPivBarIdx] = 1;
            aHPivHighs[nHPivs] = H[curPivBarIdx];
            aHPivIdxs[nHPivs] = curPivBarIdx;
            nHPivs++;
        }
    }
}

curBar = (BarCount - 1);
candIdx = 0;
candPrc = 0;
lastLPIdx = aLPivIdxs[0];
lastLPL = aLPivLows[0];
lastHPIdx = aHPivIdxs[0];
lastHPH = aHPivHighs[0];
if (lastLPIdx > lastHPIdx)
{
    candIdx = curBar - aHHVBars[curBar];
    candPrc = aHHV[curBar];
    if (lastHPH < candPrc AND candIdx > lastLPIdx AND candIdx < curBar)
    {
        aHPivs[candIdx] = 1;
        for (j = 0; j < nHPivs; j++)
        {
            aHPivHighs[nHPivs - j] = aHPivHighs[nHPivs - (j + 1)];
            aHPivIdxs[nHPivs - j] = aHPivIdxs[nHPivs - (j + 1)];
        }
        aHPivHighs[0] = candPrc;
        aHPivIdxs[0] = candIdx;
        nHPivs++;
    }
}

else
{
    candIdx = curBar - aLLVBars[curBar];
    candPrc = aLLV[curBar];
    if (lastLPL > candPrc AND candIdx > lastHPIdx AND candIdx < curBar)
    {
        aLPivs[candIdx] = 1;
        for (j = 0; j < nLPivs; j++)
        {
            aLPivLows[nLPivs - j] = aLPivLows[nLPivs - (j + 1)];
            aLPivIdxs[nLPivs - j] = aLPivIdxs[nLPivs - (j + 1)];
        }
        aLPivLows[0] = candPrc;
        aLPivIdxs[0] = candIdx;
        nLPivs++;
    }
}

for (k = 0; k < nHPivs; k++)
{
    _TRACE("High pivot no. " + k + " at barindex: " + aHPivIdxs[k] + ", " + WriteVal(ValueWhen(BarIndex() == aHPivIdxs[k], DateTime(), 1), formatDateTime) + ", " + aHPivHighs[k]);
}

a1 = aHPivs == 1;
a2 = aLPivs == 1;

Para = ParamToggle("Plot Parallel Lines","Off,On");
ColorS= ParamColor("Support",colorLime);
ColorR= ParamColor("Resistance",colorRed);
x = Cum(1);
s1 = L;
s11 = H;

pS = a2 == 1;
endt = SelectedValue(ValueWhen(ps,x,1));//LastValue
startt = SelectedValue(ValueWhen(ps,x,2));
ends = SelectedValue(ValueWhen(ps,S1,1));
starts = SelectedValue(ValueWhen(ps,S1,2));
dtS = endt - startt;
aS = (endS - startS) / dtS;
bS = endS;
trendlineS = aS *(x - endt) + bS;
g3 = IIf(x > startt -1 , trendlineS,  Null);

//Pi = 3.14159265 * atan( 1 );
//SlopeAngles = atan( trendlineS ) * ( 180 / Pi );

////////////////////////////////////////////////////
           Lowline=Ends-starts;//support
//////////////////////////////////////////////////////
g3Up = Lowline > 0;
g3Dn = Lowline < 0;

Trends = IIf( g3Up, colorBlack, iif( g3Dn, colorred, colorWhite ) ); 
Plot(g3, "", Trends , styleLine+styleThick);

pR = a1 == 1;
endt1 = SelectedValue(ValueWhen(pr,x,1));
startt1 = SelectedValue(ValueWhen(pr,x,2));
endr = SelectedValue(ValueWhen(pr,S11,1));
startr = SelectedValue(ValueWhen(pr,S11,2));
dtR = endt1 - startt1;
aR = (endR - startR) / dtR;
bR = endR;
trendlineR = aR *(x - endt1) + bR;
g4 = IIf(x > startT1 -1, trendlineR,  Null);

//Pi = 3.14159265 * atan( 1 );
//SlopeAngler = atan( g4 ) * ( 180 / Pi );
//////////////////////////////////////////////////
        Highline=endr-startr;//resistance
/////////////////////////////////////////////////
g4Up = highline > 0;
g4Dn = highline < 0;

Trendr = IIf( g4Up, colorBlack, iif( g4Dn, colorBlack, colorWhite ) ); 
Plot(g4, "", Trendr , styleLine+styleThick);

_SECTION_END();


_SECTION_BEGIN("BB");
P = ParamField("Price field",-1);
Periods = Param("Periods", 20, 2, 100, 1 );
Width = Param("Width", 2, 0, 10, 0.05 );
Color = ParamColor("Color", colorCustom10 );
Style = ParamStyle("Style", styleLine | styleNoLabel ) | styleNoLabel;
Plot( bbt = BBandTop( P, Periods, Width ), "BBTop" + _PARAM_VALUES(), Color, Style ); 
Plot( bbb = BBandBot( P, Periods, Width ), "BBBot" + _PARAM_VALUES(), Color, Style ); 
PlotOHLC( bbt, bbt, bbb, bbb, "", ColorBlend( Color, GetChartBkColor(), 0.8 ), styleNoLabel | styleCloud | styleNoRescale, Null, Null, Null, -1 );
_SECTION_END();

_SECTION_BEGIN("MA5");
P = ParamField("Price field",-1);
Periods = Param("Periods", 5, 2, 200, 1 );
Plot( MA( P, Periods ), _DEFAULT_NAME(), ParamColor( "Color", colorDarkRed ), ParamStyle("Style", styleThick) );
_SECTION_END();

_SECTION_BEGIN("MA9");
P = ParamField("Price field",-1);
Periods = Param("Periods", 9, 2, 200, 1 );
Plot( MA( P, Periods ), _DEFAULT_NAME(), ParamColor( "Color", colorRed ), ParamStyle("Style", styleThick) );
_SECTION_END();

_SECTION_BEGIN("MA20");
P = ParamField("Price field",-1);
Periods = Param("Periods", 20, 2, 200, 1 );
Plot( MA( P, Periods ), _DEFAULT_NAME(), ParamColor( "Color", colorGreen ), ParamStyle("Style", styleThick) );
_SECTION_END();

_SECTION_BEGIN("MA50");
P = ParamField("Price field",-1);
Periods = Param("Periods", 50, 2, 200, 1 );
Plot( MA( P, Periods ), _DEFAULT_NAME(), ParamColor( "Color", colorCustom12 ), ParamStyle("Style", styleThick) );
_SECTION_END();

_SECTION_BEGIN("MA100");
P = ParamField("Price field",-1);
Periods = Param("Periods", 100, 2, 200, 1 );
Plot( MA( P, Periods ), _DEFAULT_NAME(), ParamColor( "Color", colorBlue ), ParamStyle("Style", styleThick) );
_SECTION_END();

_SECTION_BEGIN("MA200");
P = ParamField("Price field",-1);
Periods = Param("Periods", 200, 2, 200, 1 );
Plot( MA( P, Periods ), _DEFAULT_NAME(), ParamColor( "Color", colorTurquoise ), ParamStyle("Style", styleThick) );
_SECTION_END();

_SECTION_BEGIN("Price");
SetChartOptions(0,chartShowArrows|chartShowDates);
_N(Title = StrFormat("{{NAME}} - {{INTERVAL}} {{DATE}} Open %g, Hi %g, Lo %g, Close %g (%.1f%%) {{VALUES}}", O, H, L, C, SelectedValue( ROC( C, 1 ) ) ));
Plot( C, "Close", ParamColor("Color", colorDefault ), styleNoTitle | ParamStyle("Style") | GetPriceStyle() ); 
_SECTION_END();

_SECTION_BEGIN("Open == High AND Open == Low Exploration");

DOpen = TimeFrameGetPrice( "O", inDaily, 0 ); // gives you Todays Open price.
DHigh = TimeFrameGetPrice( "H", inDaily, 0 ); // gives you Todays High price.
DLow = TimeFrameGetPrice( "L", inDaily, 0 ); // gives you Todays High price.
DClose = TimeFrameGetPrice( "C", inDaily, 0 ); // gives you Todays High price.

NoseBody = DClose - DOpen;
NoseLength = DHigh - Dlow;
DK1 = NoseBody / NoseLength <= 0.33;
DK2 = 1-(Min(DOpen,DClose)-Dlow)/NoseLength < 0.4;
DK3=Ref(C,-1) < Ref(O,-1);
DK4=V>100000;
DK5=C>5;

DK6=V<0.8*MA(V,20);
//DK6=V>0.8*MA(V,10);
//Buy = DK1 AND DK2 AND DK3 AND DK4 AND DK5 AND DK6; //newday AND
Buy =  DK2 AND DK3 AND DK4 AND DK5 AND DK6; //newday AND
B = Buy;
Shapes = ParamToggle("Plot Shapes","Off,On",1);
Buyshape = Param("Buy Shape Typ",1,0,50,1);
SellShape = Param("Sell Shape Typ",2,0,50,1);
Buyshapecolor = ParamColor("Buy Shape Color",colorBrightGreen);
Sellshapecolor = ParamColor("Sell Shape Color",colorRed);




PlotShapes(Buy*Buyshape*Shapes,Buyshapecolor,0,L,-15);



AlertIf (B, "Buy", "Buy at: "+C+" Alert",0, 1+2+4+8,1 );
 
PlotShapes((Buy*30),IIf(Buy,colorBlack,colorRed) );  
   


Filter = NOT GroupID()==253;
Filter = Buy;
_SECTION_END();

_SECTION_BEGIN("Dieukienmuaban");
filter = 1;
// Khoi luong
		dk1 = V > 10000 & V < 100000000;
		dk2 = C >1 & C < 200;
		chenhlechgia = (C - O)/(O * 0.01);
		AddColumn(V,"Khoi Luong",1.0);
		AddColumn(chenhlechgia,"Chenh lech gia (%)",1.2,IIf(chenhlechgia>0, colorGreen, colorRed));
	
// Duong RSI
		rsi14 = RSI(14);
		dk3 = rsi14 > 0;
		AddColumn(rsi14,"RSI",1.2);

// Duong CCI	
		cci20 = CCI(20);
		dk4 = cci20 > -100;
		AddColumn(cci20, "CCI",1.2);
	 
// Duong MFI
		mfi14 = MFI(14);
		dk5 = mfi14 > 0;
		AddColumn(mfi14, "MFI",1.2);
	
// Duong MACD
		dk6 = MACD(12,26) > SIGNAL(12,26,9);

// Duong DMI
		dk7 = PDI(14) > MDI(14);
		DMI = PDI(14) - MDI(14);
		AddColumn(DMI, "DMI", 1.2, IIf(DMI > 0,colorGreen,colorRed));
		
	
// Duong MA
		ma5 = MA(C,5);
		ma9 = MA(C,9);
		ma20 = MA(C,20);
		dk8 = C > ma20;
		dk9 = ma5 > ma9;
		dk10 = ma5 > ma20;
	
// Duong MA20 doc len
		ma20quakhu = Ref(C,-20);
		dk99 = C > ma20quakhu;
		AddColumn(ma20quakhu,"ma20quakhu",1.2);
		
		
ma5 = MA(C,5); // gia trung bình dong cua 5 ngay
ma10 = MA(C,10);
ma20 = MA(C,20);


dk11 = Cross(ma5,ma10);

vol5 = MA(V,20);


giamua = Ref(O,1);
giaban = Ref(O,4);
Lailo = (giaban - giamua)/(giamua*0.01);

AddColumn(giamua,"gia mua",1.2);
AddColumn(giaban,"gia bans",1.2);
AddColumn(Lailo,"LaiLo",1.2,IIf(Lailo > 0,colorGreen,colorRed));

Filter = dk1 & dk2 & dk5 & dk6 & dk3 & dk4 & dk8 & dk9 & dk10 & dk99;
Buy = dk1 & dk2 & dk5 & dk6 & dk3 & dk4 & dk8 & dk9 & dk10 & dk99;

_SECTION_BEGIN("Chi so tai chinh");
FS=Param("Font Size",35,11,100,1);
GfxSetTextColor( ParamColor("Color",colorBlack) );
Hor=Param("Horizontal Position",20,1,1200,1);
Ver=Param("Vertical Position",1,1,1,1);
GfxSelectFont("Times New Roman", 9, 600, bold =True, underline = False, True );


ma20 = MA(C,20);

GfxTextOut("H : " + H ,Hor+1420, Ver+20 );         /////////////////
GfxTextOut("C : " + C ,Hor+1420, Ver+50 );         /////////////////
GfxTextOut("L : " + L ,Hor+1420, Ver+80 );         /////////////////

GfxTextOut("P/B: " + Prec(  Close / GetFnData( "BookValuePerShare" ),2), Hor+1550, Ver+80 );   /////////////////////////

textma20 = NumToStr(ma20);
sma20 = StrExtract(textma20,0,'.');
nma20 = StrToNum(sma20);
strma20 = StrFormat("%g",nma20) ;
GfxTextOut("MA:" + textma20 ,Hor+1550, Ver+50 );  //////////////////



FS=Param("Font Size",35,11,100,1);
GfxSetTextColor( ParamColor("Color",colorBlack) );
Hor=Param("Horizontal Position",20,1,1200,1);
Ver=Param("Vertical Position",1,1,1,1);
GfxSelectFont("Times New Roman", 9, 600, bold =True, underline = False, True );
MAV = MA(V,21);
Voltb = (V/MAV)*10000;
text = NumToStr(Voltb);
s = StrExtract(text,0,'.');
n = 0.01*StrToNum(s);
str = StrFormat("%g",n) + " " + " % ";

GfxTextOut("%V: " + str ,Hor+1420, Ver+110 );

Giatri = (V*C)/10000;
text2 = NumToStr(Giatri)  ;
s2 = StrExtract(text2,0,'.');
n2 = 0.01*StrToNum(s2);
str2 = StrFormat("%g",n2) + " " + " ty ";

GfxTextOut("Giatri: " + str2 ,Hor+1420, Ver+140 );   ////////////////////////



Giatri100 = (10000*Giatri)/Voltb;
text100 = NumToStr(Giatri100)  ;
s100 = StrExtract(text100,0,'.');
n100 = 0.01*StrToNum(s100);
str100 = StrFormat("%g",n100) + " " + " ty ";

GfxTextOut("GiatriTB: " + str100 ,Hor+1600, Ver+140 );   ////////////////////////








w12RS = 0.4*((C)/Ref(C,-96));
w24RS = 0.2*((C)/Ref(C,-168));
w36RS = 0.2*((C)/Ref(C,-252));
w48RS = 0.2*((C)/Ref(C,-336));
RS = w12RS + w24RS + w36RS + w48RS;

text3 = NumToStr(RS)  ;


GfxTextOut("RS" + text3 ,Hor+1420, Ver+80 );    ////////////////////
GfxSelectFont("Times New Roman", 20, 600, bold =True, underline = False, True );

//GfxSelectFont("Time news roman", Status("pxheight")/6 );
GfxSetTextAlign(6 );// center alignment
GfxSetOverlayMode(0) ;                                               //An hien thong tin
//GfxSetTextColor( ColorRGB( 200, 200, 200 ) );
GfxSetTextColor( ColorHSB( 42, 42, 42 ) );
GfxSetBkMode(0); // transparent
GfxTextOut( Name(), Status("pxwidth")/2, Status("pxheight")/15 );
GfxSelectFont("Times New Roman", 9, 600, bold =True, underline = False, True );

//GfxSelectFont("Time news roman", Status("pxheight")/30 );
GfxTextOut(FullName(), Status("pxwidth")/2, Status("pxheight")/6 );
GfxSelectFont("Times New Roman", 9, 600, bold =True, underline = False, True );

//GfxSelectFont("Time news roman", Status("pxheight")/25 );
GfxTextOut("Market: " + MarketID(1), Status("pxwidth")/2, Status("pxheight")/5 );


Variable = WriteIf(C<Ref(C,-1),"Down",WriteIf(C>Ref(C,-1),"Up","Neutral"));
Rise=SelectedValue(C>Ref(C,-1));
Fall=SelectedValue(C<Ref(C,-1));
//neutral=SelectedValue(C==p);
//radius = 0.01 * Status("pxheight");
//textoffset = 1 * radius;
//GfxSelectFont("Time news roman", 12 , 500);
//GfxSetOverlayMode(0);
Color=IIf(rise,colorGreen,IIf(Fall,colorRed,ColorRGB(234,199,0)));
GfxSetTextColor(color);
//GfxTextOut("%Gia :"+ Variable , textoffset , Status("pxheight")-30 );

chenhlechgia = ((C - Ref(C,-1))/Ref(C,-1))*10000;

text1 = NumToStr(chenhlechgia)  ;
s1 = StrExtract(text1,0,'.');
n1 = 0.01*StrToNum(s1);
str1 = StrFormat("%g",n1) + "" + "% ";

GfxTextOut("%Gia : " + str1 ,Hor+1320, Ver+30 );
//GfxSetTextColor(colorBlue);


GfxTextOut(Name() ,Hor+1320, Ver+60 );


Filter = 1;
_SECTION_END();


//_SECTION_BEGIN("ZOOM");
//pxb = Status( "pxchartbottom" );
//idSlider = 100;
//function CreateGUI()
//{
//    if ( GuiSlider( idSlider, 2300,pxb-20,400,25, notifyEditChange ) == guiNew ) 
//     
//     {
//        // init values on control creation
//       GuiSetValue( idSlider, 50 );
//        GuiSetRange( idSlider, 0, 100, 1, 1 );
//     }
//}
//CreateGUI();
//GraphXSpace = guigetvalue(idslider);
//_SECTION_END(); 
