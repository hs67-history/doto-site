
# Ubuntu 26.04でNotepad++の日本語メニュー文字化けを修正する

作業日：2026年10月30日  
分類：Linux備忘録 / Ubuntu / Wine / Notepad++

## はじめに

Ubuntu 26.04にSnap版Notepad++をインストールしたところ、メニューの日本語が「□□□□」と表示される問題が発生した。

調査の結果、Snap版Notepad++が通常のWineとは別の環境を使用していることが分かった。

今回は、問題の調査から解決までを記録する。

## 1. 使用環境

- OS：Ubuntu 26.04
- アプリケーション：Notepad++ 8.9.6.4
- インストール方式：Snap
- 実行環境：Wine
- 日本語フォント：Noto Sans CJK JP

## 2. 日本語フォントを確認

Ubuntuに日本語フォントが存在するか確認した。

```bash
fc-list :lang=ja family | sort -u
```

Noto Sans CJK JP、IPAゴシック、IPA明朝などがインストールされていた。

## 3. ロケールを確認

```bash
locale
```

結果：

```text
LANG=ja_JP.UTF-8
LC_CTYPE="ja_JP.UTF-8"
LC_ALL=
```

日本語ロケールに問題はなかった。

## 4. 通常のWine設定を変更

```bash
winecfg
```

デスクトップ統合のメニューフォントをTahomaからNoto Sans CJK JPに変更したが、改善しなかった。

通常のWine環境にフォント代替設定を追加しても改善しなかった。

## 5. Snap版であることを確認

```bash
snap list | grep -i notepad
```

結果：

```text
notepad-plus-plus  8.9.6.4  444  latest/stable  mmtrt  -
```

Notepad++がSnap版であることが判明した。

## 6. Snap版Wineの場所を確認

```bash
find ~/snap/notepad-plus-plus -maxdepth 6 -type d -name drive_c
```

結果：

```text
/home/hs67/snap/notepad-plus-plus/common/.wine/drive_c
```

通常のWineとは異なる環境が使用されていた。

## 7. 日本語フォントの設定を追加

Notepad++を終了してから、次のコマンドを実行した。

```bash
WINEPREFIX="$HOME/snap/notepad-plus-plus/common/.wine" wine reg add 'HKCU\Software\Wine\Fonts\Replacements' /v 'MS UI Gothic' /t REG_SZ /d 'Noto Sans CJK JP' /f
```

実行結果：

```text
reg: 操作は正常に完了しました
```

Snap版Notepad++のWine環境に、MS UI GothicをNoto Sans CJK JPへ置き換える設定を登録できた。

## 8. Notepad++を再起動

```bash
snap run notepad-plus-plus
```

再起動すると、メニューが正常な日本語表示になった。

**文字化けの修正に成功した。**

## まとめ

今回の原因は、Snap版Notepad++が通常のWineとは別の設定環境を使用していたことにあったと考えられる。

LinuxでWindowsアプリケーションを使用するときは、インストール方式とWineの設定場所を確認することが重要だと分かった。

---

道東に暮らして — Linux備忘録
