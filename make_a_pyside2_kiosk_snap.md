# 製作一個 PySide2 Kiosk snap


## 準備

在 Ubuntu Desktop 上安裝 Mir:
```sh
$ sudo apt-add-repository --update ppa:mir-team/release
$ sudo apt install mir-demos mir-graphics-drivers-desktop
```

然後安裝 Snapcraft:
```sh
$ sudo snap install snapcraft --classic
```

<br>

## 在 Ubuntu Desktop 上進行測試

建立一個專案資料夾：
```sh
$ mkdir mir-kiosk-pyside2-example
$ cd mir-kiosk-pyside2-example
```

在專案資料夾裡建立一個 `src` 資料夾：
```sh
$ mkdir src
```

然後把 PySide2 app 的原始碼檔案放到 `src` 資料夾裡。例如：

(以下是範例程式的內容)

1. `app.py`
    ```python
    #!/usr/bin/env python3

    import sys, time
    from PySide2.QtCore import *
    from PySide2.QtWidgets import *

    class MyMDIApp(QMainWindow):
        def __init__(self):
            QMainWindow.__init__(self)
            workspace = QMdiArea()
            for i in range(8):
                textEdit = QTextEdit()
                textEdit.setPlainText("Dummy Text " * 100)
                textEdit.setWindowTitle("Document %i" % i)
                workspace.addSubWindow(textEdit)

            self.setCentralWidget(workspace)
            self.setWindowFlags(Qt.FramelessWindowHint)
            self.setGeometry(300, 300, 800, 600)
            self.show()

    def main():
        # Exception Handling
        try:
            myApp = QApplication(sys.argv)
            myMDIApp = MyMDIApp()
            myMDIApp.show()
            myApp.exec_()
            sys.exit(0)
        except NameError:
            print("Name Error:", sys.exc_info()[1])
        except SystemExit:
            print("Closing Window...")
        except Exception:
            print(sys.exc_info()[1])

    if __name__ =='__main__':
        main()
    ```
2. `__init__.py`
    ```python
    __version__ = '0.1.0'
    ```
3. `description.txt`
    ```
    Background
    ----------
    
    this is a example app
    ```

把 `app.py`、`__init__.py`、`description.txt` 這三個檔案放到 `src` 資料夾裡。

<br>

接著在專案資料夾裡建立一個 `snap` 資料夾：
```sh
$ mkdir snap
```

然後在 `snap` 資料夾裡建立一個文字檔 `snapcraft.yaml`，內容如下：

```yaml
name: mir-kiosk-pyside2-example
version: '0.1'
summary: PySide2 example kiosk
description: PySide2 example kiosk, using Wayland
base: core20
confinement: strict
grade: devel

apps:
  mir-kiosk-pyside2-example:
    command: bin/mir-kiosk-pyside2-example

plugs:
  opengl:
  wayland:

layout:
  /etc/glvnd:
    bind: $SNAP/etc/glvnd
  /etc/fonts:
    bind: $SNAP/etc/fonts
  /usr/share/fonts:
    bind: $SNAP/usr/share/fonts
  /etc/xdg:
    bind: $SNAP/etc/xdg
  /usr/share/X11/xkb:
    bind: $SNAP/usr/share/X11/xkb
  /usr/share/glvnd:
    bind: $SNAP/usr/share/glvnd
  /usr/lib/${SNAPCRAFT_ARCH_TRIPLET}/dri:
    bind: $SNAP/usr/lib/${SNAPCRAFT_ARCH_TRIPLET}/dri

environment:
  PYTHONPATH: $SNAP/pyside/
  LD_LIBRARY_PATH: ${LD_LIBRARY_PATH}:${SNAP}/pyside/PySide2/Qt/lib:${SNAP}/usr/lib/:${SNAP}/usr/lib/${SNAPCRAFT_ARCH_TRIPLET}/
  XCOMPOSEFILE: $SNAP/usr/share/X11/locale/en_US.UTF-8/Compose
  # Qt Platform to Wayland
  #QT_QPA_PLATFORM: wayland
  #QT_DEBUG_PLUGINS: 1

parts:
  mir-kiosk-pyside2-example:
    plugin: python
    source: .

  pyside2:
    plugin: nil
    override-build: |
      snapcraftctl build
      mkdir -p ${SNAPCRAFT_PART_INSTALL}/pyside
      echo "Get latest release..."
      pip3 install --no-cache-dir pyside2 -t ${SNAPCRAFT_PART_INSTALL}/pyside
      rm -r ${SNAPCRAFT_PART_INSTALL}/pyside/PySide2/examples
    build-packages:
      - python3-pip
      - wget
      - jq
    stage-packages:
      - libnss3
      - libxcomposite1
      - libxcursor1
      - libxtst6
      - libxrandr2
      - libasound2
      - libglu1-mesa
      - libgles2-mesa
      - fontconfig
      - libxkbcommon0
      - libxkbcommon-x11-0
      - libxi6
      - libegl1
      - shared-mime-info
      - libgdk-pixbuf2.0-0
      - locales-all
      - libxslt1.1
      - xfonts-base
      - xfonts-scalable
      - libxcb-xinerama0
      - libxcb-icccm4
      - libxcb-image0
      - libxcb-keysyms1
      - libxcb-randr0
      - libxcb-render-util0
      - libxcb-shape0
      - libwayland-cursor0

  mesa:
    plugin: nil
    stage-packages:
      - libgl1-mesa-dri
      - libwayland-egl1-mesa
      - libglu1-mesa
```

然後用 snapcraft 指令來建置一個 snap 檔案：
```sh
$ snapcraft --use-lxd
```
snapcraft 指令會在專案資料夾下產生一個 `mir-kiosk-pyside2-example_0.1_amd64.snap` 檔案。

<br>

來測試 `mir-kiosk-pyside2-example_0.1_amd64.snap` 檔案：

啟動 `miral-kiosk`：
```sh
$ miral-kiosk&
```

安裝 `mir-kiosk-pyside2-example_0.1_amd64.snap`：
```sh
$ sudo snap install --dangerous ./mir-kiosk-pyside2-example_0.1_amd64.snap --devmode
```

把 `mir-kiosk-pyside2-example` 連接到 `Wayland`：
```sh
$ snap connect mir-kiosk-pyside2-example:wayland
```

然後運行 `mir-kiosk-pyside2-example`：
```sh
$ snap run mir-kiosk-pyside2-example
```

運行後可以發現雖然 PySide2 程式有成功執行，但是沒有跑在 `Mir On X` 的視窗裡，而是跑在一個獨立的視窗...

讓我們來修正這個問題。

把 `snapcraft.yaml` 裡 `QT_QPA_PLATFORM: wayland` 那行的註解拿掉：
```yaml
environment:
  ...
  ...
  
  QT_QPA_PLATFORM: wayland
```

重新建置 snap 檔案：
```sh
$ snapcraft --use-lxd
```

再次安裝與運行 snap：

```sh
$ miral-kiosk&
$ sudo snap install --dangerous ./mir-kiosk-pyside2-example_0.1_amd64.snap --devmode
$ snap connect mir-kiosk-pyside2-example:wayland
$ snap run mir-kiosk-pyside2-example
```

執行後出現以下錯誤訊息：
```
Failed to create wl_display (No such file or directory)
qt.qpa.plugin: Could not load the Qt platform plugin "wayland" in "" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, wayland-egl, wayland, wayland-xcomposite-egl, wayland-xcomposite-glx, webgl, xcb.
```

讓我們來修正這個問題。

使用 Mir 團隊提供的 `mir-kiosk-snap-launch` 工具，可以幫助我們解決這個問題，修改 `snapcraft.yaml`，在 `parts:` 下方加入 `mir-kiosk-snap-launch:`，內容如下：

```yaml
parts:
  ...
  ...

  mir-kiosk-snap-launch:
    plugin: dump
    source: https://github.com/MirServer/mir-kiosk-snap-launch.git
    override-build: $SNAPCRAFT_PART_BUILD/build-with-plugs.sh opengl wayland
    stage-packages:
      - inotify-tools
```

然後把 `apps:` 的內容修改成： 
```yaml
apps:
  mir-kiosk-pyside2-example:
    command-chain:
      - bin/wayland-launch
    command: bin/mir-kiosk-pyside2-example
```

重新建置、安裝與執行：
```sh
$ snapcraft --use-lxd
$ miral-kiosk&
$ sudo snap install --dangerous ./mir-kiosk-pyside2-example_0.1_amd64.snap --devmode
$ snap connect mir-kiosk-pyside2-example:wayland
$ snap run mir-kiosk-pyside2-example
```

現在可以看到 mir-kiosk-pyside2-example app 成功跑在 `Mir On X` 裡了。

<br>

## 部署到 Ubuntu Core 20

先用 ssh 登入到你的 Ubuntu Core 機器，然後在 Ubuntu Core 裡安裝 `mir-kiosk`：
```sh
$ snap install mir-kiosk
```

因為一般期望的 Kiosk 程式是要在作業系統啟動後自動執行，且當程式當掉時，必須要有自動重啟的功能，所以需要再次修改 `snapcraft.yaml`，把 `apps:` 的內容修改成： 
```yaml
apps:
  daemon:
    command-chain:
      - bin/run-daemon
      - bin/wayland-launch
    command: bin/mir-kiosk-pyside2-example
    daemon: simple
    restart-condition: always
  
  mir-kiosk-pyside2-example:
    command-chain:
      - bin/wayland-launch
    command: bin/mir-kiosk-pyside2-example
```
 
這樣就可以讓這個程式以 `daemon (守護行程)` 的方式運行。

重新建置 snap 檔案：
```sh
$ snapcraft --use-lxd
```

透過 scp 指令把 snap 檔案複製到 Ubuntu Core 上：
```sh
$ scp mir-kiosk-pyside2-example_0.1_amd64.snap <user>@<ip-address>:~
```
`<user>` 是你的 Ubuntu Core 帳號，`<ip-address>` 是 Ubuntu Core 機器的 IP 位址。

用 ssh 登入你的 Ubuntu Core 機器，然後安裝 `mir-kiosk-pyside2-example_0.1_amd64.snap`：
```sh
$ snap install --dangerous ./mir-kiosk-pyside2-example_0.1_amd64.snap
```

現在應該可以看到 PySide2 app 在 Ubuntu Core 成功執行的畫面了！

如果沒有成功執行，可以用 `sudo snap logs -f mir-kiosk-pyside2-example` 查看 logs，找出問題出在哪裡。

如果想停止程式運行，可以用 `snap stop mir-kiosk-pyside2-example`。

<br>

## 注意事項

- 用 pip 安裝的 PySide2 有自己的 .so 檔案，所以不需要在 `stage-packages:` 加入 `qtwayland` 與 `qt5-default` 等套件。
- snap run 加上 --shell 參數可以用來查看 snap 內部檔案結構，例如：
    ```sh
    $ snap run --shell mir-kiosk-pyside2-example
    $ ls $SNAP
    ```
    這樣會列出程式主要資料夾裡的檔案。

- 因為我把環境變數 `PYTHONPATH` 設定成 `$SNAP/pyside/`，所以可以在 snap run --shell 模式裡用 `ls $SNAP/pyside/PySide2` 與 `ls $SNAP/pyside/PySide2/Qt/lib` 來查看 PySide2 有那些 .so 檔案。
- 需要把 `${SNAP}/pyside/PySide2/Qt/lib` 加到環境變數 `LD_LIBRARY_PATH` 裡，這樣 PySide2 程式執行時，才能找到正確的 .so 檔案。
- 如果把 `snapcraft.yaml` 裡的 `#QT_DEBUG_PLUGINS: 1` 這行註解拿掉，可以用來查看 Qt 執行時的 logs，除錯時很有用。

<br>

## 完整的範例程式碼

- [https://github.com/kyumdbot/mir-kiosk-pyside2-example](https://github.com/kyumdbot/mir-kiosk-pyside2-example)

