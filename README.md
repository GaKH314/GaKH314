main.mp3
TITLEJA:さいたま2000 - GaKH314バージョン
SUBTITLEJA:  
BPM:120
WAVE:さいたま2000.ogg
OFFSET:-2.077
DEMOSTART:37.793
SCOREMODE:2
MAKER:GaKH314
SONGVOL:100
SEVOL:100
SIDE:1
LIFE:0
GAME:Taiko
HEADSCROLL:0.8
BGIMAGE:bg.png
BGMOVIE:bg.avi
MOVIEOFFSET:1.5
TAIKOWEBSKIN:dir ../song_skins,name yokai,song static,stage  ,don fastscroll
COURSE:Edit
LEVEL:7
BALLOON:6,15,3,30,6,15,3
SCOREINIT:1000000
SCOREDIFF:500000
STYLE:Single
#MEASURE 4/4
1000100011101010,
#MEASURE 2/4
0212,
#SCROLL 4
30
#SCROLL -4
11201022112010,
5800,
0,
#BRANCHSTART r,1,2
#N
1111,
#E
22202220,
#M
12121212,
11111111,
,
#SECTION
#BRANCHSTART p,50,75
#BRANCHEND
11111111,
,
#BRANCHSTART p,50,75
#BRANCHEND
11111111,

using System;
using System.Collections.Generic;
using System.Text;
using System.Runtime.InteropServices;
using System.Drawing;
using System.Diagnostics;
using System.IO;
using System.Globalization;
using System.Threading;
using System.Text.RegularExpressions;
using FDK;
using FDK.ExtensionMethods;
using TJAPlayer3;

namespace TJAPlayer3
{
    internal class CDTX : CActivity
    {
        // 定数

        public enum E種別 { DTX, GDA, G2D, BMS, BME, SMF }

        // クラス

        public class CAVI : IDisposable
        {
            public CAvi avi;
            private bool bDispose済み;
            public int n番号;
            public string strコメント文 = "";
            public string strファイル名 = "";

            public void OnDeviceCreated()
            {
                #region [ strAVIファイル名の作成。]
                //-----------------
                string strAVIファイル名;
                if (!string.IsNullOrEmpty(TJAPlayer3.DTX.PATH_WAV))
                    strAVIファイル名 = TJAPlayer3.DTX.PATH_WAV + this.strファイル名;
                else
                    strAVIファイル名 = TJAPlayer3.DTX.strフォルダ名 + this.strファイル名;
                //-----------------
                #endregion

                if (!File.Exists(strAVIファイル名))
                {
                    Trace.TraceWarning("ファイルが存在しません。({0})({1})", this.strコメント文, strAVIファイル名);
                    this.avi = null;
                    return;
                }

                // AVI の生成。

                try
                {
                    this.avi = new CAvi(strAVIファイル名);
                    Trace.TraceInformation("動画を生成しました。({0})({1})({2}frames)", this.strコメント文, strAVIファイル名, this.avi.GetMaxFrameCount());
                }
                catch (Exception e)
                {
                    Trace.TraceError(e.ToString());
                    Trace.TraceError("動画の生成に失敗しました。({0})({1})", this.strコメント文, strAVIファイル名);
                    this.avi = null;
                }
            }
            public override string ToString()
            {
                return string.Format("CAVI{0}: File:{1}, Comment:{2}", CDTX.tZZ(this.n番号), this.strファイル名, this.strコメント文);
            }

            #region [ IDisposable 実装 ]
            //-----------------
            public void Dispose()
            {
                if (this.bDispose済み)
                    return;

                if (this.avi != null)
                {
                    #region [ strAVIファイル名 の作成。 ]
                    //-----------------
                    string strAVIファイル名;
                    if (!string.IsNullOrEmpty(TJAPlayer3.DTX.PATH_WAV))
                        strAVIファイル名 = TJAPlayer3.DTX.PATH_WAV + this.strファイル名;
                    else
                        strAVIファイル名 = TJAPlayer3.DTX.strフォルダ名 + this.strファイル名;
                    //-----------------
                    #endregion

                    this.avi.Dispose();
                    this.avi = null;

                    Trace.TraceInformation("動画を解放しました。({0})({1})", this.strコメント文, strAVIファイル名);
                }

                this.bDispose済み = true;
            }
            //-----------------
            #endregion
        }
        public class CAVIPAN
        {
            public int nAVI番号;
            public int n移動時間ct;
            public int n番号;
            public Point pt動画側開始位置 = new Point(0, 0);
            public Point pt動画側終了位置 = new Point(0, 0);
            public Point pt表示側開始位置 = new Point(0, 0);
            public Point pt表示側終了位置 = new Point(0, 0);
            public Size sz開始サイズ = new Size(0, 0);
            public Size sz終了サイズ = new Size(0, 0);

            public override string ToString()
            {
                return string.Format("CAVIPAN{0}: AVI:{14}, 開始サイズ:{1}x{2}, 終了サイズ:{3}x{4}, 動画側開始位置:{5}x{6}, 動画側終了位置:{7}x{8}, 表示側開始位置:{9}x{10}, 表示側終了位置:{11}x{12}, 移動時間:{13}ct",
                    CDTX.tZZ(this.n番号),
                    this.sz開始サイズ.Width, this.sz開始サイズ.Height,
                    this.sz終了サイズ.Width, this.sz終了サイズ.Height,
                    this.pt動画側開始位置.X, this.pt動画側開始位置.Y,
                    this.pt動画側終了位置.X, this.pt動画側終了位置.Y,
                    this.pt表示側開始位置.X, this.pt表示側開始位置.Y,
                    this.pt表示側終了位置.X, this.pt表示側終了位置.Y,
                    this.n移動時間ct,
                    CDTX.tZZ(this.nAVI番号));
            }
        }
        public class CDirectShow : IDisposable
        {
            public FDK.CDirectShow dshow;
            private bool bDispose済み;
            public int n番号;
            public string strコメント文 = "";
            public string strファイル名 = "";

            public void OnDeviceCreated()
            {
                #region [ str動画ファイル名の作成。]
                //-----------------
                string str動画ファイル名;
                if (!string.IsNullOrEmpty(TJAPlayer3.DTX.PATH_WAV))
                    str動画ファイル名 = TJAPlayer3.DTX.PATH_WAV + this.strファイル名;
                else
                    str動画ファイル名 = TJAPlayer3.DTX.strフォルダ名 + this.strファイル名;
                //-----------------
                #endregion

                if (!File.Exists(str動画ファイル名))
                {
                    Trace.TraceWarning("ファイルが存在しません。({0})({1})", this.strコメント文, str動画ファイル名);
                    this.dshow = null;
                }

                // AVI の生成。

                try
                {
                    this.dshow = new FDK.CDirectShow(TJAPlayer3.stage選曲.r確定されたスコア.ファイル情報.フォルダの絶対パス + this.strファイル名, TJAPlayer3.app.WindowHandle, true);
                    Trace.TraceInformation("DirectShowを生成しました。({0})({1})({2}byte)", this.strコメント文, str動画ファイル名, this.dshow.nデータサイズbyte);
                }
                catch (Exception e)
                {
                    Trace.TraceError(e.ToString());
                    Trace.TraceError("DirectShowの生成に失敗しました。({0})({1})", this.strコメント文, str動画ファイル名);
                    this.dshow = null;
                }
            }
            public override string ToString()
            {
                return string.Format("CAVI{0}: File:{1}, Comment:{2}", CDTX.tZZ(this.n番号), this.strファイル名, this.strコメント文);
            }

            #region [ IDisposable 実装 ]
            //-----------------
            public void Dispose()
            {
                if (this.bDispose済み)
                    return;

                if (this.dshow != null)
                {
                    #region [ strAVIファイル名 の作成。 ]
                    //-----------------
                    string str動画ファイル名;
                    if (!string.IsNullOrEmpty(TJAPlayer3.DTX.PATH_WAV))
                        str動画ファイル名 = TJAPlayer3.DTX.PATH_WAV + this.strファイル名;
                    else
                        str動画ファイル名 = TJAPlayer3.DTX.strフォルダ名 + this.strファイル名;
                    //-----------------
                    #endregion

                    this.dshow.Dispose();
                    this.dshow = null;

                    Trace.TraceInformation("動画を解放しました。({0})({1})", this.strコメント文, str動画ファイル名);
                }

                this.bDispose済み = true;
            }
            //-----------------
            #endregion
        }

        public class CBPM
        {
            public double dbBPM値;
            public double bpm_change_time;
            public double bpm_change_bmscroll_time;
            public int bpm_change_course;
            public int n内部番号;
            public int n表記上の番号;

            public override string ToString()
            {
                StringBuilder builder = new StringBuilder(0x80);
                if (this.n内部番号 != this.n表記上の番号)
                {
                    builder.Append(string.Format("CBPM{0}(内部{1})", CDTX.tZZ(this.n表記上の番号), this.n内部番号));
                }
                else
                {
                    builder.Append(string.Format("CBPM{0}", CDTX.tZZ(this.n表記上の番号)));
                }
                builder.Append(string.Format(", BPM:{0}", this.dbBPM値));
                return builder.ToString();
            }
        }
        public class CSCROLL
        {
            public double dbSCROLL値;
            public double dbSCROLL値Y;
            public int n内部番号;
            public int n表記上の番号;

            public override string ToString()
            {
                StringBuilder builder = new StringBuilder(0x80);
                if (this.n内部番号 != this.n表記上の番号)
                {
                    builder.Append(string.Format("CSCROLL{0}(内部{1})", CDTX.tZZ(this.n表記上の番号), this.n内部番号));
                }
                else
                {
                    builder.Append(string.Format("CSCROLL{0}", CDTX.tZZ(this.n表記上の番号)));
                }
                builder.Append(string.Format(", SCROLL:{0}", this.dbSCROLL値));
                return builder.ToString();
            }
        }
        /// <summary>
        /// 判定ライン移動命令
        /// </summary>
		public class CJPOSSCROLL
        {
            public double db移動時間;
            public int n移動距離px;
            public int n移動方向; //移動方向は0(左)、1(右)の2つだけ。
            public int n内部番号;
            public int n表記上の番号;

            public override string ToString()
            {
                StringBuilder builder = new StringBuilder(0x80);
                if (this.n内部番号 != this.n表記上の番号)
                {
                    builder.Append(string.Format("CJPOSSCROLL{0}(内部{1})", CDTX.tZZ(this.n表記上の番号), this.n内部番号));
                }
                else
                {
                    builder.Append(string.Format("CJPOSSCROLL{0}", CDTX.tZZ(this.n表記上の番号)));
                }
                builder.Append(string.Format(", JPOSSCROLL:{0}", this.db移動時間));
                return builder.ToString();
            }
        }

        public class CDELAY
        {
            public int nDELAY値; //格納時にはmsになっているため、doubleにはしない。
            public int n内部番号;
            public int n表記上の番号;
            public double delay_time;
            public double delay_bmscroll_time;
            public double delay_bpm;
            public int delay_course;

            public override string ToString()
            {
                StringBuilder builder = new StringBuilder(0x80);
                if (this.n内部番号 != this.n表記上の番号)
                {
                    builder.Append(string.Format("CDELAY{0}(内部{1})", CDTX.tZZ(this.n表記上の番号), this.n内部番号));
                }
                else
                {
                    builder.Append(string.Format("CDELAY{0}", CDTX.tZZ(this.n表記上の番号)));
                }
                builder.Append(string.Format(", DELAY:{0}", this.nDELAY値));
                return builder.ToString();
            }
        }

        public class CBRANCH
        {
            public int n分岐の種類; //0:精度分岐 1:連打分岐 2:スコア分岐 3:大音符のみの精度分岐
            public double n条件数値A;
            public double n条件数値B;
            public double db分岐時間;
            public double db分岐時間ms;
            public double db判定時間;
            public double dbBMScrollTime;
            public double dbBPM;
            public double dbSCROLL;
            public int n現在の小節;
            public int n命令時のChipList番号;

            public int n表記上の番号;
            public int n内部番号;

            public override string ToString()
            {
                StringBuilder builder = new StringBuilder(0x80);
                if (this.n内部番号 != this.n表記上の番号)
                {
                    builder.Append(string.Format("CBRANCH{0}(内部{1})", CDTX.tZZ(this.n表記上の番号), this.n内部番号));
                }
                else
                {
                    builder.Append(string.Format("CBRANCH{0}", CDTX.tZZ(this.n表記上の番号)));
                }
                builder.Append(string.Format(", BRANCH:{0}", this.n分岐の種類));
                return builder.ToString();
            }
        }


        public class CChip : IComparable<CDTX.CChip>, ICloneable
        {
            public bool bHit;
            public bool b可視 = true;
            public bool bShow;
            public bool bBranch = false;
            public double dbチップサイズ倍率 = 1.0;
            public double db実数値;
            public double dbBPM;
            public double dbSCROLL;
            public double dbSCROLL_Y;
            public int nコース;
            public int nSenote;
            public int nState;
            public int nRollCount;
            public int nBalloon;
            public int nProcessTime;
            public int nスクロール方向;
            public int n描画優先度; //(特殊)現状連打との判断目的で使用
            public ENoteState eNoteState;
            public EAVI種別 eAVI種別;
            public E楽器パート e楽器パート = E楽器パート.UNKNOWN;
            public int nチャンネル番号;
            public STDGBVALUE<int> nバーからの距離dot;
            public STDGBVALUE<int> nバーからのノーツ末端距離dot;
            public int n整数値;
            public int n整数値_内部番号;
            public int n総移動時間;
            public int n透明度 = 0xff;
            public int n発声位置;
            public double db発声位置;  // 発声時刻を格納していた変数のうちの１つをfloat型からdouble型に変更。(kairera0467)
            public double fBMSCROLLTime;
            public double fBMSCROLLTime_end;
            public int n発声時刻ms;
            public double db発声時刻ms;
            public int nノーツ終了位置;
            public int nノーツ終了時刻ms;
            public int nノーツ出現時刻ms;
            public int nノーツ移動開始時刻ms;
            public int n分岐回数;
            public int n連打音符State;
            public int nLag;                // 2011.2.1 yyagi
            public CDTX.CAVI rAVI;
            public CDTX.CAVIPAN rAVIPan;
            public CDTX.CDirectShow rDShow;
            public double db発声時刻;
            public double db判定終了時刻;//連打系音符で使用
            public double dbProcess_Time;
            public int nPlayerSide;
            public bool bGOGOTIME = false; //2018.03.11 k1airera0467 ゴーゴータイム内のチップであるか
            public int nList上の位置;
            public bool IsFixedSENote;
            public bool IsHitted = false;
            public bool bBPMチップである
            {
                get
                {
                    if (this.nチャンネル番号 == 3 || this.nチャンネル番号 == 8)
                    {
                        return true;
                    }
                    else
                    {
                        return false;
                    }
                }
            }
            public bool bWAVを使うチャンネルである
            {
                get
                {
                    switch (this.nチャンネル番号)
                    {
                        case 0x01:
                            return true;
                    }
                    return false;
                }
            }
            public bool b自動再生音チャンネルである
            {
                get
                {
                    int num = this.nチャンネル番号;
                    if ((((num != 1) && ((0x61 > num) || (num > 0x69))) && ((0x70 > num) || (num > 0x79))) && ((0x80 > num) || (num > 0x89)))
                    {
                        return ((0x90 <= num) && (num <= 0x92));
                    }
                    return true;
                }
            }
            public bool b演奏終了後も再生が続くチップである;	// #32248 2013.10.14 yyagi
            public CCounter RollDelay; // 18.9.22 AioiLight Add 連打時に赤くなるやつのタイマー
            public CCounter RollInputTime; // 18.9.22 AioiLight Add  連打入力後、RollDelayが作動するまでのタイマー
            public int RollEffectLevel; // 18.9.22 AioiLight Add 連打時に赤くなるやつの度合い

            public CChip()
            {
                this.nバーからの距離dot = new STDGBVALUE<int>()
                {
                    Drums = 0,
                    Guitar = 0,
                    Bass = 0,
                };
            }
            public void t初期化()
            {
                this.bBranch = false;
                this.nチャンネル番号 = 0;
                this.n整数値 = 0; //整数値をList上の番号として用いる。
                this.n整数値_内部番号 = 0;
                this.db実数値 = 0.0;
                this.n発声位置 = 0;
                this.db発声位置 = 0.0D;
                this.n発声時刻ms = 0;
                this.db発声時刻ms = 0.0D;
                this.fBMSCROLLTime = 0;
                this.nノーツ終了位置 = 0;
                this.nノーツ終了時刻ms = 0;
                this.n描画優先度 = 0;
                this.nLag = -999;
                this.b演奏終了後も再生が続くチップである = false;
                this.nList上の位置 = 0;
                this.dbチップサイズ倍率 = 1.0;
                this.bHit = false;
                this.b可視 = true;
                this.e楽器パート = E楽器パート.UNKNOWN;
                this.n透明度 = 0xff;
                this.nバーからの距離dot.Drums = 0;
                this.nバーからの距離dot.Guitar = 0;
                this.nバーからの距離dot.Bass = 0;
                this.nバーからの距離dot.Taiko = 0;
                this.nバーからのノーツ末端距離dot.Drums = 0;
                this.nバーからのノーツ末端距離dot.Guitar = 0;
                this.nバーからのノーツ末端距離dot.Bass = 0;
                this.nバーからのノーツ末端距離dot.Taiko = 0;
                this.n総移動時間 = 0;
                this.dbBPM = 120.0;
                this.nスクロール方向 = 0;
                this.dbSCROLL = 1.0;
                this.dbSCROLL_Y = 0.0f;
            }
            public override string ToString()
            {

                //2016.10.07 kairera0467 近日中に再編成予定
                string[] chToStr =
                {
                    //システム
					"??", "バックコーラス", "小節長変更", "BPM変更", "??", "??", "??", "??",
                    "BPM変更(拡張)", "??", "??", "??", "??", "??", "??", "??",

                    //太鼓1P(移動予定)
					"??", "ドン", "カツ", "ドン(大)", "カツ(大)", "連打", "連打(大)", "ふうせん連打",
                    "連打終点", "芋", "ドン(手)", "カッ(手)", "??", "??", "??", "AD-LIB",

                    //太鼓予備
					"??", "??", "??", "??", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??",

                    //太鼓予備
					"??", "??", "??", "??", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??",

                    //太鼓予備
					"??", "??", "??", "??", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??", 

                    //システム
					"小節線", "拍線", "??", "??", "AVI", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??", 

                    //システム(移動予定)
					"SCROLL", "DELAY", "ゴーゴータイム開始", "ゴーゴータイム終了", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??",

                    "??", "??", "??", "??", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??",

                    "??", "??", "??", "??", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??", 

                    //太鼓1P、システム(現行)
					"??", "??", "??", "太鼓_赤", "太鼓_青", "太鼓_赤(大)", "太鼓_青(大)", "太鼓_黄",
                    "太鼓_黄(大)", "太鼓_風船", "太鼓_連打末端", "太鼓_芋", "??", "SCROLL", "ゴーゴータイム開始", "ゴーゴータイム終了",

                    "??", "??", "??", "??", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "太鼓 AD-LIB",

                    "??", "??", "??", "??", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??",

                    "??", "??", "??", "??", "0xC4", "0xC5", "0xC6", "??",
                    "??", "??", "0xCA", "??", "??", "??", "??", "0xCF", 

                    //システム(現行)
					"0xD0", "??", "??", "??", "??", "??", "??", "??",
                    "??", "??", "ミキサー追加", "ミキサー削除", "DELAY", "譜面分岐リセット", "譜面分岐アニメ", "譜面分岐内部処理", 

                    //システム(現行)
					"小節線ON/OFF", "分岐固定", "判定枠移動", "", "", "", "", "",
                    "", "", "", "", "", "", "", "",

                    "0xF0", "歌詞", "??", "SUDDEN", "??", "??", "??", "??",
                    "??", "??", "??", "??", "??", "??", "??", "??", "譜面終了"
                };
                return string.Format("CChip: 位置:{0:D4}.{1:D3}, 時刻{2:D6}, Ch:{3:X2}({4}), Pn:{5}({11})(内部{6}), Pd:{7}, Sz:{8}, BMScroll:{9}, Auto:{10}, コース:{11}",
                    this.n発声位置 / 384, this.n発声位置 % 384,
                    this.n発声時刻ms,
                    this.nチャンネル番号, chToStr[this.nチャンネル番号],
                    this.n整数値, this.n整数値_内部番号,
                    this.db実数値,
                    this.dbチップサイズ倍率,
                    this.fBMSCROLLTime,
                    this.b自動再生音チャンネルである,
                    this.nコース,
                    CDTX.tZZ(this.n整数値));
            }
            /// <summary>
            /// チップの再生長を取得する。現状、WAVチップとBGAチップでのみ使用可能。
            /// </summary>
            /// <returns>再生長(ms)</returns>
            public int GetDuration()
            {
                int nDuration = 0;

                if (this.bWAVを使うチャンネルである)       // WAV
                {
                    CDTX.CWAV wc;
                    TJAPlayer3.DTX.listWAV.TryGetValue(this.n整数値_内部番号, out wc);
                    if (wc == null)
                    {
                        nDuration = 0;
                    }
                    else
                    {
                        nDuration = (wc.rSound[0] == null) ? 0 : wc.rSound[0].n総演奏時間ms;
                    }
                }
                else if (this.nチャンネル番号 == 0x54) // AVI
                {
                    if (this.rAVI != null && this.rAVI.avi != null)
                    {
                        int dwRate = (int)this.rAVI.avi.dwレート;
                        int dwScale = (int)this.rAVI.avi.dwスケール;
                        nDuration = (int)(1000.0f * dwScale / dwRate * this.rAVI.avi.GetMaxFrameCount());
                    }
                }

                double _db再生速度 = (TJAPlayer3.DTXVmode.Enabled) ? TJAPlayer3.DTX.dbDTXVPlaySpeed : TJAPlayer3.DTX.db再生速度;
                return (int)(nDuration / _db再生速度);
            }

            #region [ IComparable 実装 ]
            //-----------------

            private static readonly byte[] n優先度 = new byte[] {
                5, 5, 3, 7, 5, 5, 5, 5, 3, 5, 5, 5, 5, 5, 5, 5, //0x00
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0x10
		        7, 7, 7, 7, 7, 7, 7, 7, 5, 5, 5, 5, 5, 5, 5, 5, //0x20
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0x30
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0x40
		        9, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0x50
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0x60
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0x70
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0x80
		        5, 5, 5, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 9, 9, 9, //0x90
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0xA0
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0xB0
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0xC0
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 3, 4, 4, //0xD0
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0xE0
		        5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, //0xF0
		    };

            public int CompareTo(CDTX.CChip other)
            {
                // まずは位置で比較。

                //BGMチップだけ発声位置
                //if( this.nチャンネル番号 == 0x01 || this.nチャンネル番号 == 0x02 )
                //{
                //    if( this.n発声位置 < other.n発声位置 )
                //        return -1;

                //    if( this.n発声位置 > other.n発声位置 )
                //        return 1;
                //}

                //if( this.n発声位置 < other.n発声位置 )
                //    return -1;

                //if( this.n発声位置 > other.n発声位置 )
                //    return 1;

                //譜面解析メソッドV4では発声時刻msで比較する。
                var n発声時刻msCompareToResult = 0;
                n発声時刻msCompareToResult = this.n発声時刻ms.CompareTo(other.n発声時刻ms);
                if (n発声時刻msCompareToResult != 0)
                {
                    return n発声時刻msCompareToResult;
                }

                n発声時刻msCompareToResult = this.db発声時刻ms.CompareTo(other.db発声時刻ms);
                if (n発声時刻msCompareToResult != 0)
                {
                    return n発声時刻msCompareToResult;
                }

                // 位置が同じなら優先度で比較。
                return n優先度[this.nチャンネル番号].CompareTo(n優先度[other.nチャンネル番号]);
            }
            //-----------------
            #endregion
            /// <summary>
            /// shallow copyです。
            /// </summary>
            /// <returns></returns>
            public object Clone()
            {
                return MemberwiseClone();
            }
        }
        public class CWAV : IDisposable
        {
            public bool bBGMとして使う;
            public List<int> listこのWAVを使用するチャンネル番号の集合 = new List<int>(16);
            public int nチップサイズ = 100;
            public int n位置;
            public long[] n一時停止時刻 = new long[TJAPlayer3.ConfigIni.nPoliphonicSounds];    // 4
            public int SongVol = CSound.DefaultSongVol;
            public LoudnessMetadata? SongLoudnessMetadata = null;
            public int n現在再生中のサウンド番号;
            public long[] n再生開始時刻 = new long[TJAPlayer3.ConfigIni.nPoliphonicSounds];    // 4
            public int n内部番号;
            public int n表記上の番号;
            public CSound[] rSound = new CSound[TJAPlayer3.ConfigIni.nPoliphonicSounds];     // 4
            public string strコメント文 = "";
            public string strファイル名 = "";
            public bool bBGMとして使わない
            {
                get
                {
                    return !this.bBGMとして使う;
                }
                set
                {
                    this.bBGMとして使う = !value;
                }
            }
            public bool bIsBassSound = false;
            public bool bIsGuitarSound = false;
            public bool bIsDrumsSound = false;
            public bool bIsSESound = false;
            public bool bIsBGMSound = false;

            public override string ToString()
            {
                var sb = new StringBuilder(128);

                if (this.n表記上の番号 == this.n内部番号)
                {
                    sb.Append(string.Format("CWAV{0}: ", CDTX.tZZ(this.n表記上の番号)));
                }
                else
                {
                    sb.Append(string.Format("CWAV{0}(内部{1}): ", CDTX.tZZ(this.n表記上の番号), this.n内部番号));
                }
                sb.Append(
                    $"{nameof(SongVol)}:{this.SongVol}, {nameof(LoudnessMetadata.Integrated)}:{this.SongLoudnessMetadata?.Integrated}, {nameof(LoudnessMetadata.TruePeak)}:{this.SongLoudnessMetadata?.TruePeak}, 位置:{this.n位置}, サイズ:{this.nチップサイズ}, BGM:{(this.bBGMとして使う ? 'Y' : 'N')}, File:{this.strファイル名}, Comment:{this.strコメント文}");

                return sb.ToString();
            }

            #region [ Dispose-Finalize パターン実装 ]
            //-----------------
            public void Dispose()
            {
                this.Dispose(true);
                GC.SuppressFinalize(this);
            }
            public void Dispose(bool bManagedリソースの解放も行う)
            {
                if (this.bDisposed済み)
                    return;

                if (bManagedリソースの解放も行う)
                {
                    for (int i = 0; i < TJAPlayer3.ConfigIni.nPoliphonicSounds; i++) // 4
                    {
                        if (this.rSound[i] != null)
                            TJAPlayer3.Sound管理.tサウンドを破棄する(this.rSound[i]);
                        this.rSound[i] = null;

                        if ((i == 0) && TJAPlayer3.ConfigIni.bLog作成解放ログ出力)
                            Trace.TraceInformation("サウンドを解放しました。({0})({1})", this.strコメント文, this.strファイル名);
                    }
                }

                this.bDisposed済み = true;
            }
            ~CWAV()
            {
                this.Dispose(false);
            }
            //-----------------
            #endregion

            #region [ private ]
            //-----------------
            private bool bDisposed済み;
            //-----------------
            #endregion
        }

        public class DanSongs
        {
            public CTexture TitleTex;
            public CTexture SubTitleTex;
            public string Title;
            public string SubTitle;
            public string FileName;
            public string Genre;
            public int ScoreInit;
            public int ScoreDiff;
            public static int Number;
            public CWAV Wave;

            public DanSongs()
            {
                Number++;
            }
        }


        // 構造体

        public struct STLANEINT
        {
            public int HH;
            public int SD;
            public int BD;
            public int HT;
            public int LT;
            public int CY;
            public int FT;
            public int HHO;
            public int RD;
            public int LC;
            public int LP;
            public int LBD;

            public int Drums
            {
                get
                {
                    return this.HH + this.SD + this.BD + this.HT + this.LT + this.CY + this.FT + this.HHO + this.RD + this.LC + this.LP + this.LBD;
                }
            }
            public int Guitar;
            public int Bass;
            public int Taiko_Red;
            public int Taiko_Blue;

            public int this[int index]
            {
                get
                {
                    switch (index)
                    {
                        case 0:
                            return this.HH;

                        case 1:
                            return this.SD;

                        case 2:
                            return this.BD;

                        case 3:
                            return this.HT;

                        case 4:
                            return this.LT;

                        case 5:
                            return this.CY;

                        case 6:
                            return this.FT;

                        case 7:
                            return this.HHO;

                        case 8:
                            return this.RD;

                        case 9:
                            return this.LC;

                        case 10:
                            return this.LP;

                        case 11:
                            return this.LBD;

                        case 12:
                            return this.Guitar;

                        case 13:
                            return this.Bass;

                        case 14:
                            return this.Taiko_Red;

                        case 15:
                            return this.Taiko_Blue;
                    }
                    throw new IndexOutOfRangeException();
                }
                set
                {
                    if (value < 0)
                    {
                        throw new ArgumentOutOfRangeException();
                    }
                    switch (index)
                    {
                        case 0:
                            this.HH = value;
                            return;

                        case 1:
                            this.SD = value;
                            return;

                        case 2:
                            this.BD = value;
                            return;

                        case 3:
                            this.HT = value;
                            return;

                        case 4:
                            this.LT = value;
                            return;

                        case 5:
                            this.CY = value;
                            return;

                        case 6:
                            this.FT = value;
                            return;

                        case 7:
                            this.HHO = value;
                            return;

                        case 8:
                            this.RD = value;
                            return;

                        case 9:
                            this.LC = value;
                            return;

                        case 10:
                            this.LP = value;
                            return;

                        case 11:
                            this.LBD = value;
                            return;

                        case 12:
                            this.Guitar = value;
                            return;

                        case 13:
                            this.Bass = value;
                            return;

                        case 14:
                            this.Taiko_Red = value;
                            return;

                        case 15:
                            this.Taiko_Blue = value;
                            return;
                    }
                    throw new IndexOutOfRangeException();
                }
            }
        }
        public struct STRESULT
        {
            public string SS;
            public string S;
            public string A;
            public string B;
            public string C;
            public string D;
            public string E;

            public string this[int index]
            {
                get
                {
                    switch (index)
                    {
                        case 0:
                            return this.SS;

                        case 1:
                            return this.S;

                        case 2:
                            return this.A;

                        case 3:
                            return this.B;

                        case 4:
                            return this.C;

                        case 5:
                            return this.D;

                        case 6:
                            return this.E;
                    }
                    throw new IndexOutOfRangeException();
                }
                set
                {
                    switch (index)
                    {
                        case 0:
                            this.SS = value;
                            return;

                        case 1:
                            this.S = value;
                            return;

                        case 2:
                            this.A = value;
                            return;

                        case 3:
                            this.B = value;
                            return;

                        case 4:
                            this.C = value;
                            return;

                        case 5:
                            this.D = value;
                            return;

                        case 6:
                            this.E = value;
                            return;
                    }
                    throw new IndexOutOfRangeException();
                }
            }
        }
        public struct STチップがある
        {
            public bool Drums;
            public bool Guitar;
            public bool Bass;

            public bool HHOpen;
            public bool Ride;
            public bool LeftCymbal;
            public bool OpenGuitar;
            public bool OpenBass;

            public bool Branch;

            public bool this[int index]
            {
                get
                {
                    switch (index)
                    {
                        case 0:
                            return this.Drums;

                        case 1:
                            return this.Guitar;

                        case 2:
                            return this.Bass;

                        case 3:
                            return this.HHOpen;

                        case 4:
                            return this.Ride;

                        case 5:
                            return this.LeftCymbal;

                        case 6:
                            return this.OpenGuitar;

                        case 7:
                            return this.OpenBass;

                        case 8:
                            return this.Branch;
                    }
                    throw new IndexOutOfRangeException();
                }
                set
                {
                    switch (index)
                    {
                        case 0:
                            this.Drums = value;
                            return;

                        case 1:
                            this.Guitar = value;
                            return;

                        case 2:
                            this.Bass = value;
                            return;

                        case 3:
                            this.HHOpen = value;
                            return;

                        case 4:
                            this.Ride = value;
                            return;

                        case 5:
                            this.LeftCymbal = value;
                            return;

                        case 6:
                            this.OpenGuitar = value;
                            return;

                        case 7:
                            this.OpenBass = value;
                            return;

                        case 8:
                            this.Branch = value;
                            return;
                    }
                    throw new IndexOutOfRangeException();
                }
            }
        }

        public class CLine
        {
            public int n小節番号;
            public int n文字数;
            public double db発声時刻;
            public double dbBMS時刻;
            public int nコース;
            public int nタイプ;
        }

        // プロパティ

        public int nBGMAdjust
        {
            get;
            private set;
        }
        public int nPlayerSide; //2017.08.14 kairera0467 引数で指定する
        public bool bDP譜面が存在する;
        public bool bSession譜面を読み込む;
        public bool IsDanChallenge; // 2018/8/24 段位チャレンジが存在するか否か (AioiLight)

        public string ARTIST;
        public string BACKGROUND;
        public string BACKGROUND_GR;
        public double BASEBPM;
        public double BPM;
        public STチップがある bチップがある;
        public string COMMENT;
        public double db再生速度;
        public E種別 e種別;
        public string GENRE;
        public Eジャンル eジャンル;
        public bool HIDDENLEVEL;
        public STDGBVALUE<int> LEVEL;
        public int[] LEVELtaiko = new int[(int)Difficulty.Total] { -1, -1, -1, -1, -1, -1, -1 };
        public Dictionary<int, CAVI> listAVI;
        public Dictionary<int, CAVIPAN> listAVIPAN;
        public Dictionary<int, CDirectShow> listDS;
        public Dictionary<int, CBPM> listBPM;
        public List<CChip> listChip;
        public Dictionary<int, CWAV> listWAV;
        public Dictionary<int, CSCROLL> listSCROLL;
        public Dictionary<int, CSCROLL> listSCROLL_Normal;
        public Dictionary<int, CSCROLL> listSCROLL_Expert;
        public Dictionary<int, CSCROLL> listSCROLL_Master;
        public Dictionary<int, CJPOSSCROLL> listJPOSSCROLL;
        public List<DanSongs> List_DanSongs;


        private int listSCROLL_Normal_数値管理;
        private int listSCROLL_Expert_数値管理;
        private int listSCROLL_Master_数値管理;

        private double[] dbNowSCROLL_Normal;
        private double[] dbNowSCROLL_Expert;
        private double[] dbNowSCROLL_Master;


        public Dictionary<int, CDELAY> listDELAY;
        public Dictionary<int, CBRANCH> listBRANCH;
        public STLANEINT n可視チップ数;
        public const int n最大音数 = 4;
        public const int n小節の解像度 = 384;
        public string PANEL;
        public string PATH_WAV;
        public string PREIMAGE;
        public string PREVIEW;
        public string strハッシュofDTXファイル;
        public string strファイル名;
        public string strファイル名の絶対パス;
        public string strフォルダ名;
        public string SUBTITLE;
        public string TITLE;
        public double dbDTXVPlaySpeed;
        public double dbScrollSpeed;
        public int nデモBGMオフセット;

        private int n現在の小節数 = 1;
        private bool bBarLine = true;
        private int n命令数 = 0;

        private int nNowRoll = 0;
        private int nNowRollCount = 0;

        private int[] n連打チップ_temp = new int[3];


        private int nCount = 0;

        public int nOFFSET = 0;
        private bool bOFFSETの値がマイナスである = false;
        private int nMOVIEOFFSET = 0;
        private bool bMOVIEOFFSETの値がマイナスである = false;
        private double dbNowBPM = 120.0;
        private int nDELAY = 0;
        public bool[] bHasBranch = new bool[(int)Difficulty.Total] { false, false, false, false, false, false, false };

        //分岐関連
        private int n現在の発声時刻;
        private int n現在の発声時刻ms;
        private int n現在のコース;

        private bool b最初の分岐である;
        public int[] nノーツ数 = new int[4]; //0～2:各コース 3:共通
        public int[] n風船数 = new int[4]; //0～2:各コース 3:共通
        private bool b次の小節が分岐である;
        private bool b次の分岐で数値リセット; //2018.03.16 kairera0467 SECTION処理を分岐判定と同時に行う。
        private int n文字数;
        private bool b直前の行に小節末端定義が無かった = false;
        private int n命令行のチップ番号_temp = 0;

        private List<CLine> listLine;
        private int nLineCountTemp; //分岐開始時の小節数を記録。
        private int nLineCountCourseTemp; //現在カウント中のコースを記録。

        public int n参照中の難易度 = 3;
        public int nScoreModeTmp = 99; //2017.01.28 DD
        public int[,] nScoreInit = new int[2, (int)Difficulty.Total]; //[ x, y ] x=通常or真打 y=コース
        public int[] nScoreDiff = new int[(int)Difficulty.Total]; //[y]
        public bool[,] b配点が指定されている = new bool[3, (int)Difficulty.Total]; //2017.06.04 kairera0467 [ x, y ] x=通常(Init)or真打orDiff y=コース

        private double dbBarLength;
        public float fNow_Measure_s = 4.0f;
        public float fNow_Measure_m = 4.0f;
        public double dbNowTime = 0.0;
        public double dbNowBMScollTime = 0.0;
        public double dbNowScroll = 1.0;
        public double dbNowScrollY = 0.0; //2016.08.13 kairera0467 複素数スクロール
        public double dbLastTime = 0.0; //直前の小節の開始時間
        public double dbLastBMScrollTime = 0.0;

        public int[] bBARLINECUE = new int[2]; //命令を入れた次の小節の操作を実現するためのフラグ。0 = mainflag, 1 = cuetype
        public bool b小節線を挿入している = false;

        //Normal Regular Masterにしたいけどここは我慢。
        private List<int> listBalloon_Normal;
        private List<int> listBalloon_Expert;
        private List<int> listBalloon_Master;
        private List<int> listBalloon; //旧構文用

        public List<string> listLiryc; //歌詞を格納していくリスト。スペル忘れた(ぉい

        private int listBalloon_Normal_数値管理;
        private int listBalloon_Expert_数値管理;
        private int listBalloon_Master_数値管理;

        public bool[] b譜面が存在する = new bool[(int)Difficulty.Total];

        private string[] dlmtSpace = { " " };
        private string[] dlmtEnter = { "\n" };
        private string[] dlmtCOURSE = { "COURSE:" };

        private int nスクロール方向 = 0;
        //2015.09.18 kairera0467
        //バタフライスライドみたいなアレをやりたいがために実装。
        //次郎2みたいな複素数とかは意味不明なので、方向を指定してスクロールさせることにした。
        //0:通常
        //1:上
        //2:下
        //3:右上
        //4:右下
        //5:左
        //6:左上
        //7:左下

        public string strBGIMAGE_PATH;
        public string strBGVIDEO_PATH;

        public double db出現時刻;
        public double db移動待機時刻;

        public string strBGM_PATH;
        public int SongVol;
        public LoudnessMetadata? SongLoudnessMetadata;

        public bool bHIDDENBRANCH; //2016.04.01 kairera0467 選曲画面上、譜面分岐開始前まで譜面分岐の表示を隠す
        public bool bGOGOTIME; //2018.03.11 kairera0467

        public bool IsEndedBranching; // BRANCHENDが呼び出されたかどうか
        public Dan_C[] Dan_C;

        public bool IsEnabledFixSENote;
        public int FixSENote;
        public GaugeIncreaseMode GaugeIncreaseMode;



#if TEST_NOTEOFFMODE
		public STLANEVALUE<bool> b演奏で直前の音を消音する;
//		public bool bHH演奏で直前のHHを消音する;
//		public bool bGUITAR演奏で直前のGUITARを消音する;
//		public bool bBASS演奏で直前のBASSを消音する;
#endif
        // コンストラクタ

        public CDTX()
        {
            this.nPlayerSide = 0;
            this.TITLE = "";
            this.SUBTITLE = "";
            this.ARTIST = "";
            this.COMMENT = "";
            this.PANEL = "";
            this.GENRE = "";
            this.eジャンル = Eジャンル.None;
            this.PREVIEW = "";
            this.PREIMAGE = "";
            this.BACKGROUND = "";
            this.BACKGROUND_GR = "";
            this.PATH_WAV = "";
            this.BPM = 120.0;
            STDGBVALUE<int> stdgbvalue = new STDGBVALUE<int>();
            stdgbvalue.Drums = 0;
            stdgbvalue.Guitar = 0;
            stdgbvalue.Bass = 0;
            this.LEVEL = stdgbvalue;
            this.bHIDDENBRANCH = false;
            this.db再生速度 = 1.0;
            this.bチップがある = new STチップがある();
            this.bチップがある.Drums = false;
            this.bチップがある.Guitar = false;
            this.bチップがある.Bass = false;
            this.bチップがある.HHOpen = false;
            this.bチップがある.Ride = false;
            this.bチップがある.LeftCymbal = false;
            this.bチップがある.OpenGuitar = false;
            this.bチップがある.OpenBass = false;
            this.strファイル名 = "";
            this.strフォルダ名 = "";
            this.strファイル名の絶対パス = "";
            this.n無限管理WAV = new int[36 * 36];
            this.n無限管理BPM = new int[36 * 36];
            this.n無限管理PAN = new int[36 * 36];
            this.n無限管理SIZE = new int[36 * 36];
            this.listBalloon_Normal_数値管理 = 0;
            this.listBalloon_Expert_数値管理 = 0;
            this.listBalloon_Master_数値管理 = 0;
            this.nRESULTIMAGE用優先順位 = new int[7];
            this.nRESULTMOVIE用優先順位 = new int[7];
            this.nRESULTSOUND用優先順位 = new int[7];

            #region [ 2011.1.1 yyagi GDA->DTX変換テーブル リファクタ後 ]
            STGDAPARAM[] stgdaparamArray = new STGDAPARAM[] {		// GDA->DTX conversion table
				new STGDAPARAM("TC", 0x03), new STGDAPARAM("BL", 0x02), new STGDAPARAM("GS", 0x29),
                new STGDAPARAM("DS", 0x30), new STGDAPARAM("FI", 0x53), new STGDAPARAM("HH", 0x11),
                new STGDAPARAM("SD", 0x12), new STGDAPARAM("BD", 0x13), new STGDAPARAM("HT", 0x14),
                new STGDAPARAM("LT", 0x15), new STGDAPARAM("CY", 0x16), new STGDAPARAM("G1", 0x21),
                new STGDAPARAM("G2", 0x22), new STGDAPARAM("G3", 0x23), new STGDAPARAM("G4", 0x24),
                new STGDAPARAM("G5", 0x25), new STGDAPARAM("G6", 0x26), new STGDAPARAM("G7", 0x27),
                new STGDAPARAM("GW", 0x28), new STGDAPARAM("01", 0x61), new STGDAPARAM("02", 0x62),
                new STGDAPARAM("03", 0x63), new STGDAPARAM("04", 0x64), new STGDAPARAM("05", 0x65),
                new STGDAPARAM("06", 0x66), new STGDAPARAM("07", 0x67), new STGDAPARAM("08", 0x68),
                new STGDAPARAM("09", 0x69), new STGDAPARAM("0A", 0x70), new STGDAPARAM("0B", 0x71),
                new STGDAPARAM("0C", 0x72), new STGDAPARAM("0D", 0x73), new STGDAPARAM("0E", 0x74),
                new STGDAPARAM("0F", 0x75), new STGDAPARAM("10", 0x76), new STGDAPARAM("11", 0x77),
                new STGDAPARAM("12", 0x78), new STGDAPARAM("13", 0x79), new STGDAPARAM("14", 0x80),
                new STGDAPARAM("15", 0x81), new STGDAPARAM("16", 0x82), new STGDAPARAM("17", 0x83),
                new STGDAPARAM("18", 0x84), new STGDAPARAM("19", 0x85), new STGDAPARAM("1A", 0x86),
                new STGDAPARAM("1B", 0x87), new STGDAPARAM("1C", 0x88), new STGDAPARAM("1D", 0x89),
                new STGDAPARAM("1E", 0x90), new STGDAPARAM("1F", 0x91), new STGDAPARAM("20", 0x92),
                new STGDAPARAM("B1", 0xA1), new STGDAPARAM("B2", 0xA2), new STGDAPARAM("B3", 0xA3),
                new STGDAPARAM("B4", 0xA4), new STGDAPARAM("B5", 0xA5), new STGDAPARAM("B6", 0xA6),
                new STGDAPARAM("B7", 0xA7), new STGDAPARAM("BW", 0xA8), new STGDAPARAM("G0", 0x20),
                new STGDAPARAM("B0", 0xA0)
            };
            this.stGDAParam = stgdaparamArray;
            #endregion
            this.nBGMAdjust = 0;
            this.nPolyphonicSounds = TJAPlayer3.ConfigIni.nPoliphonicSounds;
            this.dbDTXVPlaySpeed = 1.0f;

            //this.nScoreModeTmp = 1;
            for (int y = 0; y < (int)Difficulty.Total; y++)
            {
                this.nScoreInit[0, y] = 300;
                this.nScoreInit[1, y] = 1000;
                this.nScoreDiff[y] = 120;
                this.b配点が指定されている[0, y] = false;
                this.b配点が指定されている[1, y] = false;
                this.b配点が指定されている[2, y] = false;
            }

            this.bBarLine = true;

            this.dbBarLength = 1.0;

            this.b最初の分岐である = true;
            this.b次の小節が分岐である = false;

            this.SongVol = CSound.DefaultSongVol;
            this.SongLoudnessMetadata = null;

            GaugeIncreaseMode = GaugeIncreaseMode.Normal;

#if TEST_NOTEOFFMODE
			this.bHH演奏で直前のHHを消音する = true;
			this.bGUITAR演奏で直前のGUITARを消音する = true;
			this.bBASS演奏で直前のBASSを消音する = true;
#endif

            Thread.CurrentThread.CurrentCulture = CultureInfo.InvariantCulture; // Change default culture to invariant, fixes (Purota)
            Dan_C = new Dan_C[3];
        }
        public CDTX(string str全入力文字列)
            : this()
        {
            this.On活性化();
            this.t入力_全入力文字列から(str全入力文字列);
        }
        public CDTX(string strファイル名, bool bヘッダのみ)
            : this()
        {
            this.On活性化();
            this.t入力(strファイル名, bヘッダのみ);
        }
        public CDTX(string str全入力文字列, double db再生速度, int nBGMAdjust)
            : this()
        {
            this.On活性化();
            this.t入力_全入力文字列から(str全入力文字列, str全入力文字列, db再生速度, nBGMAdjust);
        }
        public CDTX(string strファイル名, bool bヘッダのみ, double db再生速度, int nBGMAdjust)
            : this()
        {
            this.On活性化();
            this.t入力(strファイル名, bヘッダのみ, db再生速度, nBGMAdjust, 0, 0, false);
        }
        public CDTX(string strファイル名, bool bヘッダのみ, double db再生速度, int nBGMAdjust, int nReadVersion)
            : this()
        {
            this.On活性化();
            this.t入力(strファイル名, bヘッダのみ, db再生速度, nBGMAdjust, nReadVersion, 0, false);
        }
        public CDTX(string strファイル名, bool bヘッダのみ, double db再生速度, int nBGMAdjust, int nReadVersion, int nPlayerSide, bool bSession)
            : this()
        {
            this.On活性化();
            this.t入力(strファイル名, bヘッダのみ, db再生速度, nBGMAdjust, nReadVersion, nPlayerSide, bSession);
        }


        // メソッド

        public void tAVIの読み込み()
        {
            if (this.listAVI != null)
            {
                foreach (CAVI cavi in this.listAVI.Values)
                {
                    cavi.OnDeviceCreated();
                }
            }
            if (this.listDS != null)
            {
                foreach (CDirectShow cds in this.listDS.Values)
                {
                    cds.OnDeviceCreated();
                }
            }
            if (!this.bヘッダのみ)//&& this.b動画読み込み )
            {
                foreach (CChip chip in this.listChip)
                {
                    if (chip.nチャンネル番号 == 0x54 || chip.nチャンネル番号 == 0x5A)
                    {
                        chip.eAVI種別 = EAVI種別.Unknown;
                        chip.rAVI = null;
                        chip.rDShow = null;
                        chip.rAVIPan = null;
                        if (this.listAVIPAN.TryGetValue(chip.n整数値, out CAVIPAN cavipan))
                        {
                            if (this.listAVI.TryGetValue(cavipan.nAVI番号, out CAVI cavi) && (cavi.avi != null))
                            {
                                chip.eAVI種別 = EAVI種別.AVIPAN;
                                chip.rAVI = cavi;
                                //if( CDTXMania.ConfigIni.bDirectShowMode == true )
                                chip.rDShow = this.listDS[cavipan.nAVI番号];
                                chip.rAVIPan = cavipan;
                                continue;
                            }
                        }

                        CDirectShow ds = null;
                        if (this.listAVI.TryGetValue(chip.n整数値, out CAVI cavi2) && (cavi2.avi != null) || (this.listDS.TryGetValue(chip.n整数値, out ds) && (ds.dshow != null)))
                        {
                            chip.eAVI種別 = EAVI種別.AVI;
                            chip.rAVI = cavi2;
                            //if(CDTXMania.ConfigIni.bDirectShowMode == true)
                            chip.rDShow = ds;
                        }
                    }
                }
            }
        }

        public void tWave再生位置自動補正()
        {
            foreach (CWAV cwav in this.listWAV.Values)
            {
                this.tWave再生位置自動補正(cwav);
            }
        }
        public void tWave再生位置自動補正(CWAV wc)
        {
            if (wc.rSound[0] != null && wc.rSound[0].n総演奏時間ms >= 5000)
            {
                for (int i = 0; i < nPolyphonicSounds; i++)
                {
                    if ((wc.rSound[i] != null) && (wc.rSound[i].b再生中))
                    {
                        long nCurrentTime = CSound管理.rc演奏用タイマ.nシステム時刻ms;
                        if (nCurrentTime > wc.n再生開始時刻[i])
                        {
                            long nAbsTimeFromStartPlaying = nCurrentTime - wc.n再生開始時刻[i];
                            //Trace.TraceInformation( "再生位置自動補正: {0}, seek先={1}ms, 全音長={2}ms",
                            //    Path.GetFileName( wc.rSound[ 0 ].strファイル名 ),
                            //    nAbsTimeFromStartPlaying,
                            //    wc.rSound[ 0 ].n総演奏時間ms
                            //);
                            // wc.rSound[ i ].t再生位置を変更する( wc.rSound[ i ].t時刻から位置を返す( nAbsTimeFromStartPlaying ) );
                            wc.rSound[i].t再生位置を変更する(nAbsTimeFromStartPlaying);  // WASAPI/ASIO用
                        }
                    }
                }
            }
        }
        public void tWavの再生停止(int nWaveの内部番号)
        {
            tWavの再生停止(nWaveの内部番号, false);
        }
        public void tWavの再生停止(int nWaveの内部番号, bool bミキサーからも削除する)
        {
            if (this.listWAV.TryGetValue(nWaveの内部番号, out CWAV cwav))
            {
                for (int i = 0; i < nPolyphonicSounds; i++)
                {
                    if (cwav.rSound[i] != null && cwav.rSound[i].b再生中)
                    {
                        if (bミキサーからも削除する)
                        {
                            cwav.rSound[i].tサウンドを停止してMixerからも削除する();
                        }
                        else
                        {
                            cwav.rSound[i].t再生を停止する();
                        }
                    }
                }
            }
        }
        public void tWAVの読み込み(CWAV cwav)
        {
            string str = string.IsNullOrEmpty(this.PATH_WAV) ? this.strフォルダ名 : this.PATH_WAV;
            str = str + cwav.strファイル名;

            try
            {
                #region [ 同時発音数を、チャンネルによって変える ]

                int nPoly = nPolyphonicSounds;
                if (TJAPlayer3.Sound管理.GetCurrentSoundDeviceType() != "DirectSound") // DShowでの再生の場合はミキシング負荷が高くないため、
                {
                    // チップのライフタイム管理を行わない
                    if (cwav.bIsBassSound) nPoly = (nPolyphonicSounds >= 2) ? 2 : 1;
                    else if (cwav.bIsGuitarSound) nPoly = (nPolyphonicSounds >= 2) ? 2 : 1;
                    else if (cwav.bIsSESound) nPoly = 1;
                    else if (cwav.bIsBGMSound) nPoly = 1;
                }

                if (cwav.bIsBGMSound) nPoly = 1;

                #endregion

                for (int i = 0; i < nPoly; i++)
                {
                    try
                    {
                        cwav.rSound[i] = TJAPlayer3.Sound管理.tサウンドを生成する(str, ESoundGroup.SongPlayback);

                        if (!TJAPlayer3.ConfigIni.bDynamicBassMixerManagement)
                        {
                            cwav.rSound[i].tBASSサウンドをミキサーに追加する();
                        }

                        if (TJAPlayer3.ConfigIni.bLog作成解放ログ出力)
                        {
                            Trace.TraceInformation("サウンドを作成しました。({3})({0})({1})({2}bytes)", cwav.strコメント文, str,
                                cwav.rSound[0].nサウンドバッファサイズ, cwav.rSound[0].bストリーム再生する ? "Stream" : "OnMemory");
                        }
                    }
                    catch (Exception e)
                    {
                        cwav.rSound[i] = null;
                        Trace.TraceError("サウンドの作成に失敗しました。({0})({1})", cwav.strコメント文, str);
                        Trace.TraceError(e.ToString());
                    }
                }
            }
            catch (Exception exception)
            {
                Trace.TraceError("サウンドの生成に失敗しました。({0})({1})", cwav.strコメント文, str);
                Trace.TraceError(exception.ToString());

                for (int j = 0; j < nPolyphonicSounds; j++)
                {
                    cwav.rSound[j] = null;
                }

                //continue;
            }
        }

        public static string tZZ(int n)
        {
            if (n < 0 || n >= 36 * 36)
                return "!!";    // オーバー／アンダーフロー。

            // n を36進数2桁の文字列にして返す。

            string str = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
            return new string(new char[] { str[n / 36], str[n % 36] });
        }
        public void tギターとベースのランダム化(E楽器パート part, Eランダムモード eRandom)
        {
        }
        public void t太鼓チップのランダム化(Eランダムモード eRandom)
        {
            //2016.02.11 kairera0467
            //なんだよこのクソ実装は(怒)
            Random rnd = new System.Random();

            switch (eRandom)
            {
                case Eランダムモード.MIRROR:
                    foreach (var chip in this.listChip)
                    {
                        switch (chip.nチャンネル番号)
                        {
                            case 0x11:
                                chip.nチャンネル番号 = 0x12;
                                break;
                            case 0x12:
                                chip.nチャンネル番号 = 0x11;
                                break;
                            case 0x13:
                                chip.nチャンネル番号 = 0x14;
                                chip.nSenote = 6;
                                break;
                            case 0x14:
                                chip.nチャンネル番号 = 0x13;
