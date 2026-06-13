# このフォークについて

Reina-Sakiria が 唐突に BigscreenBeyond を 無線化できれば嬉しいよね！

そうすれば __無線__ かつ __軽量__ で __ライトハウス__ な HMD になるよねってことで試したこと、それの痕跡。

なお、その計画はほぼほぼ諦めって感じ。

## 何をしようとしたのか

Orange Pi 5 は HDMI 2.1 を DSC 込で吐き出せるそうで、そこに HDMI to DP の Active 変換ケーブル(148B-HDMI-DP-8K, チップ : LT6711GX-U3)を噛ませれば

BigscreenBeyond に画面を映せるのではないか ...という発想から始まります。

BigscreenBeyond は [EDID](#bsb-edid) を見るに DSC 込でないと絶対に受け取らないという超変態仕様なわけなんですけども、 LT6711GX-U3 の変換ケーブルは DSC をパススルーしたり、はたまた DSC をエンコード出来るというカタログスペック上の性能が見受けられるわけです。

また、 Orange Pi 5 こと RK3588S はカタログスペック上は HDMI の出力に限り DSC の出力が可能ともカタログスペック上の性能が見受けられます。

こいつら繋いだら映らないかなぁ ... ()

## orange-pi-6.1-rk35xx-rs64

ひとまず debian を Orange Pi 5 に焼いて、カーネルのビルド環境を整え、カスタムカーネルを動かせるようになったので AI (gemini 無料の範囲(Reina-Sakiria はお金持ちではないため高級な AI が使えません)) に完全に頼り切りになりながら、パッチを当ててもらったブランチです。

様々　BSB の解像度を要求されたときにだけ、検証をバイパスしたり、リンクトレーニングのタイミングを調節して死なないようにしたり ... 

そもそもこの RockChip のドライバ DSC や FRL (かなり広帯域の場合に使われる新しい方式 ... ?) を 8K のときにしか使わないみたいな、結構決め打ち実装になっているようでそこら辺を BSS 解像度の場合にのみ通すみたいなことをしながら無理やり、そこら辺のコンポーネントを叩き起こすようなことを行い ... 

Vop が幅 4096 までしか行えないそうだから、何故かうまく動かないのだけどそこら辺のフラグを無理やり立てるパッチを当てたり ...

結局、5088x2544 を通す方針は、これだけやってログに致命的そうなエラーが出なくなっても、問題なさそうでも BSB は映りませんでした

## orange-pi-6.1-rk35xx-rs64-2

少し方針転換して ... BSB にはネイティブ解像度ではない 3840x1920 を受け取れるようなので、そっちの方針に流してみようということでパッチを当て直したのがこっち。

(このブランチは、前のブランチを参考にしつつも自分で書いたパッチが半分くらい。コミットメッセージに `based on` ってい入ってるやつは私が頑張って書いた)

前と同じようにいくつかバイパスをねじ込みつつも、 FRL だけ有効化し、 DSC の方は特に何もせず、 HDMI 2.1 の帯域はギリギリだけど、そのまま出力させて、`LT6711GX-U3` の DSC エンコーダーが動くことを期待したというもの ...

でも結局動かなかった。

## どうすればいいんだろうね？

LT6711GX-U3 はカタログスペック上は BSB よりも大きな解像度を通せる(、つまるところ帯域的にはな)ので、行けるかなとは思ったんだけど、それが出来ない可能性もあるし、現状チップに対して直接の操作やログの読み取りが出来ないわけだけど、直接なにかネゴシエーションできれば出来る可能性もある ... 

どうしたものか ... こういったチップそれ単体で Dev kit のようなものってあるのだろうか ... ? (多分無い) 多分これ以上は個人じゃ出来ないのだろうか ... ?

一応 BSB をその変換チップ無しで RTX2060SP などに繋いだ時は Orange Pi 上で試した手法で画面が映ることが確認できている ... なので Linux が致命的に悪いというわけではないだろう。

結局 Orange Pi 5 からその特殊解像度の出力が出せてるのか否かすらもよくわからない。変換がうまく行ってないのか、その切り分けを行える機材も情報も資金もない ... 限界ってやつだね ()

この計画自体主目的は、 BSB の無線化 (基本的には Wifi ベース) になるわけだけど、 Orange Pi とかの arm 系 Soc じゃなくて intel embedded みたいな、もはやメインライン最新の Linux カーネルが動くようなドライバもしっかりしたものがあって DSC 込で Display Port 1.4 を出力できる小さい PC を用意できればいけるのだろうかね ... (BSB が映って、課題の領域を ユーザースペースプログラム(OpenXR など)の範囲に持ち込めれば私がどうにでも出来るでしょうし ... )

## 参考情報

### BSB-EDID

```edid
edid-decode (hex):

00 ff ff ff ff ff ff 00 09 27 34 12 d2 04 00 00
ff 20 01 04 a5 00 00 78 00 78 75 aa 55 3b b2 29
10 50 54 00 00 00 01 01 01 01 01 01 01 01 01 01
01 01 01 01 01 01 00 00 00 10 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 10 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 10 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 fc
00 42 65 79 6f 6e 64 0a 20 20 20 20 20 20 01 5b

70 20 79 07 00 22 09 28 39 54 0f 88 df 13 7f 00
3f 80 1f 00 ef 09 17 00 0e 80 01 00 1e f5 0a 08
ff 0e ff 00 3f 80 1f 00 7f 07 1b 00 0e 80 01 00
7e 00 07 3a 02 92 81 00 08 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e1 90

----------------

Block 0, Base EDID:
  EDID Structure Version & Revision: 1.4
  Vendor & Product Identification:
    Manufacturer: BIG
    Model: 4660
    Serial Number: 1234 (0x000004d2)
    Model year: 2022
  Basic Display Parameters & Features:
    Digital display
    Bits per primary color channel: 8
    DisplayPort interface
    Image size is variable
    Gamma: 2.20
    Supported color formats: RGB 4:4:4
    First detailed timing does not include the native pixel format and preferred refresh rate
  Color Characteristics:
    Red  : 0.6650, 0.3349
    Green: 0.2324, 0.6953
    Blue : 0.1611, 0.0654
    White: 0.3134, 0.3291
  Established Timings I & II: none
  Standard Timings: none
  Detailed Timing Descriptors:
    Dummy Descriptor:
    Dummy Descriptor:
    Dummy Descriptor:
    Display Product Name: 'Beyond'
  Extension blocks: 1
Checksum: 0x5b

----------------

Block 1, DisplayID Extension Block:
  Version: 2.0
  Extension Count: 0
  Display Product Primary Use Case: Head-mounted Virtual Reality (VR) display
  Video Timing Modes Type 7 - Detailed Timings Data Block:
    These timings support DSC pass-through
    DTD:  5088x2544   75.000030 Hz   0:0    192.600 kHz   1004.602000 MHz (aspect undefined, no 3D stereo, preferred)
               Hfront   64 Hsync  32 Hback   32 Hpol P
               Vfront   15 Vsync   2 Vback    7 Vpol P
    DTD:  3840x1920   90.000035 Hz   0:0    175.320 kHz    718.111000 MHz (aspect undefined, no 3D stereo)
               Hfront   64 Hsync  32 Hback  160 Hpol P
               Vfront   15 Vsync   2 Vback   11 Vpol P
  Vendor-Specific Data Block (0x7e) (VESA), OUI 3A-02-92:
    Data Structure Type: DP
    Default Colorspace and EOTF Handling: Native as specified in the Display Parameters DB
    Number of Pixels in Hor Pix Cnt Overlapping an Adjacent Panel: 0
    Multi-SST Operation: Not Supported
    Pass through timing's target DSC bits per pixel: 8.0000
  Checksum: 0xe1
Checksum: 0x90
```

### BSB-EDID(コネクタ越し)

```edid
[reina@reina-orangepi5 ~]$ sudo cat /sys/class/drm/card0-HDMI-A-1/edid | edid-decode
edid-decode (hex):

00 ff ff ff ff ff ff 00 09 27 34 12 d2 04 00 00
ff 20 01 03 80 00 00 78 00 78 75 aa 55 3b b2 29
10 50 54 00 00 00 01 01 01 01 01 01 01 01 01 01
01 01 01 01 01 01 00 00 00 10 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 10 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 10 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 fc
00 42 65 79 6f 6e 64 0a 20 20 20 20 20 20 02 80

02 03 26 f1 51 10 1f 20 05 14 04 13 12 11 03 02
16 15 07 06 01 5f 23 09 1f 07 83 01 00 00 67 03
0c 00 10 00 00 44 a3 66 00 a0 f0 70 1f 80 30 20
35 00 ba 89 21 00 00 1a 56 5e 00 a0 a0 a0 29 50
30 20 35 00 ba 89 21 00 00 1a 7c 39 00 a0 80 38
1f 40 30 20 3a 00 ba 89 21 00 00 1a a8 16 00 a0
80 38 13 40 30 20 3a 00 ba 89 21 00 00 1a 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 14

70 20 79 07 00 22 09 28 39 54 0f 88 df 13 7f 00
3f 80 1f 00 ef 09 17 00 0e 80 01 00 1e f5 0a 08
ff 0e ff 00 3f 80 1f 00 7f 07 1b 00 0e 80 01 00
7e 00 07 3a 02 92 81 00 08 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e1 90

----------------

Block 0, Base EDID:
  EDID Structure Version & Revision: 1.3
  Vendor & Product Identification:
    Manufacturer: BIG
    Model: 4660
    Serial Number: 1234 (0x000004d2)
    Model year: 2022
  Basic Display Parameters & Features:
    Digital display
    Image size is variable
    Gamma: 2.20
    Monochrome or grayscale display
  Color Characteristics:
    Red  : 0.6650, 0.3349
    Green: 0.2324, 0.6953
    Blue : 0.1611, 0.0654
    White: 0.3134, 0.3291
  Established Timings I & II: none
  Standard Timings: none
  Detailed Timing Descriptors:
    Dummy Descriptor:
    Dummy Descriptor:
    Dummy Descriptor:
    Display Product Name: 'Beyond'
  Extension blocks: 2
Checksum: 0x80

----------------

Block 1, CTA-861 Extension Block:
  Revision: 3
  Underscans IT Video Formats by default
  Basic audio support
  Supports YCbCr 4:4:4
  Supports YCbCr 4:2:2
  Native detailed modes: 1
  Video Data Block:
    VIC  16:  1920x1080   60.000000 Hz  16:9     67.500 kHz    148.500000 MHz
    VIC  31:  1920x1080   50.000000 Hz  16:9     56.250 kHz    148.500000 MHz
    VIC  32:  1920x1080   24.000000 Hz  16:9     27.000 kHz     74.250000 MHz
    VIC   5:  1920x1080i  60.000000 Hz  16:9     33.750 kHz     74.250000 MHz
    VIC  20:  1920x1080i  50.000000 Hz  16:9     28.125 kHz     74.250000 MHz
    VIC   4:  1280x720    60.000000 Hz  16:9     45.000 kHz     74.250000 MHz
    VIC  19:  1280x720    50.000000 Hz  16:9     37.500 kHz     74.250000 MHz
    VIC  18:   720x576    50.000000 Hz  16:9     31.250 kHz     27.000000 MHz
    VIC  17:   720x576    50.000000 Hz   4:3     31.250 kHz     27.000000 MHz
    VIC   3:   720x480    59.940060 Hz  16:9     31.469 kHz     27.000000 MHz
    VIC   2:   720x480    59.940060 Hz   4:3     31.469 kHz     27.000000 MHz
    VIC  22:  1440x576i   50.000000 Hz  16:9     15.625 kHz     27.000000 MHz
    VIC  21:  1440x576i   50.000000 Hz   4:3     15.625 kHz     27.000000 MHz
    VIC   7:  1440x480i   59.940060 Hz  16:9     15.734 kHz     27.000000 MHz
    VIC   6:  1440x480i   59.940060 Hz   4:3     15.734 kHz     27.000000 MHz
    VIC   1:   640x480    59.940476 Hz   4:3     31.469 kHz     25.175000 MHz
    VIC  95:  3840x2160   30.000000 Hz  16:9     67.500 kHz    297.000000 MHz
  Audio Data Block:
    Linear PCM:
      Max channels: 2
      Supported sample rates (kHz): 96 88.2 48 44.1 32
      Supported sample sizes (bits): 24 20 16
  Speaker Allocation Data Block:
    FL/FR - Front Left/Right
  Vendor-Specific Data Block (HDMI), OUI 00-0C-03:
    Source physical address: 1.0.0.0
    Maximum TMDS clock: 340 MHz
  Detailed Timing Descriptors:
    DTD 1:  3840x2160   29.980602 Hz  16:9     65.688 kHz    262.750000 MHz (698 mm x 393 mm)
                 Hfront   48 Hsync  32 Hback   80 Hpol P
                 Vfront    3 Vsync   5 Vback   23 Vpol N
    DTD 2:  2560x1440   59.950550 Hz  16:9     88.787 kHz    241.500000 MHz (698 mm x 393 mm)
                 Hfront   48 Hsync  32 Hback   80 Hpol P
                 Vfront    3 Vsync   5 Vback   33 Vpol N
    DTD 3:  2048x1080   59.989695 Hz 256:135   66.649 kHz    147.160000 MHz (698 mm x 393 mm)
                 Hfront   48 Hsync  32 Hback   80 Hpol P
                 Vfront    3 Vsync  10 Vback   18 Vpol N
    DTD 4:  2048x1080   23.901834 Hz 256:135   26.268 kHz     58.000000 MHz (698 mm x 393 mm)
                 Hfront   48 Hsync  32 Hback   80 Hpol P
                 Vfront    3 Vsync  10 Vback    6 Vpol N
Checksum: 0x14  Unused space in Extension Block: 17 bytes

----------------

Block 2, DisplayID Extension Block:
  Version: 2.0
  Extension Count: 0
  Display Product Primary Use Case: Head-mounted Virtual Reality (VR) display
  Video Timing Modes Type 7 - Detailed Timings Data Block:
    These timings support DSC pass-through
    DTD:  5088x2544   75.000030 Hz   0:0    192.600 kHz   1004.602000 MHz (aspect undefined, no 3D stereo, preferred)
               Hfront   64 Hsync  32 Hback   32 Hpol P
               Vfront   15 Vsync   2 Vback    7 Vpol P
    DTD:  3840x1920   90.000035 Hz   0:0    175.320 kHz    718.111000 MHz (aspect undefined, no 3D stereo)
               Hfront   64 Hsync  32 Hback  160 Hpol P
               Vfront   15 Vsync   2 Vback   11 Vpol P
  Vendor-Specific Data Block (0x7e) (VESA), OUI 3A-02-92:
    Data Structure Type: DP
    Default Colorspace and EOTF Handling: Native as specified in the Display Parameters DB
    Number of Pixels in Hor Pix Cnt Overlapping an Adjacent Panel: 0
    Multi-SST Operation: Not Supported
    Pass through timing's target DSC bits per pixel: 8.0000
  Checksum: 0xe1
Checksum: 0x90
```

## Original-README

Linux kernel
============

There are several guides for kernel developers and users. These guides can
be rendered in a number of formats, like HTML and PDF. Please read
Documentation/admin-guide/README.rst first.

In order to build the documentation, use ``make htmldocs`` or
``make pdfdocs``.  The formatted documentation can also be read online at:

    https://www.kernel.org/doc/html/latest/

There are various text files in the Documentation/ subdirectory,
several of them using the Restructured Text markup notation.

Please read the Documentation/process/changes.rst file, as it contains the
requirements for building and running the kernel, and information about
the problems which may result by upgrading your kernel.
