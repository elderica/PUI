# FAQ

## DynamoRioについて

### CreateWindowEXをdrwrap_replace()で置き換える場合について
以下の様な場合を考えます。

```
if (_stricmp(module_name, "USER32.dll") == 0) {
	to_wrap = (app_pc)dr_get_proc_address(info->handle, "CreateWindowEX");
	drwrap_replace(to_wrap, (app_pc)CreateWindowEX_interceptor, NULL);
}

static HWND
CreateWindowEX_interceptor(
  DWORD     dwExStyle,
  LPCSTR    lpClassName,
  LPCSTR    lpWindowName,
  DWORD     dwStyle,
  int       X,
  int       Y,
  int       nWidth,
  int       nHeight,
  HWND      hWndParent,
  HMENU     hMenu,
  HINSTANCE hInstance,
  LPVOID    lpParam)
{
	if (lpClassNameはTOOLTIPS) return 適当な戻り値;
  return CreateWindowEX(
    DWORD     dwExStyle,
    LPCSTR    lpClassName,
    LPCSTR    lpWindowName,
    DWORD     dwStyle,
    int       X,
    int       Y,
    int       nWidth,
    int       nHeight,
    HWND      hWndParent,
    HMENU     hMenu,
    HINSTANCE hInstance,
    LPVOID    lpParam);
}
```
#### 原因
省略

#### 解決方法
不明

この様な場合、CreateWindowEX_interceptor内部のCreateWindowEX呼び出しに対しても、drwrap_replaceが影響してしまい、再帰的無限ループに陥りそうである。

## WinAFLについて
### trimに時間が掛かる
winAFL実行開始からbitflip戦略を開始するまでの間に、先だってtrimが実施される。この際、10分ほど時間を要することがあった。この時のexec speedは1/sec未満である。終了後のexec speedは正常である。

#### 原因
不明

#### 解決方法
しばらく待つ。

### bit flipsから先に進まない
単純に、seedとしてinputディレクトリに用意したファイルが大き過ぎる。なるべく小さいファイルを用意すべきで、例えば画像データのパーサをFuzzingするのであれば、専用のコーパスを利用するべきである。

#### 原因
bit flips戦略の特性上、処理量はファイルサイズに比例して大きくなる。

#### 解決方法
seedファイルのサイズを小さくする。

### 実行開始後しばらくしてもlast new pathが更新されない
WinAFLを実行してから数分以内にパスが通らない場合、恐らく解析対象のアプリケーションが正しく動作せず、入力ファイルを読み込むことができていないと考えられる。また、別に考えられる理由としては、デフォルトのメモリ制限```-m```が厳し過ぎるため、プログラムの割り当てに失敗して終了していることである。他にも、入力ファイルが最低限のファイルフォーマットに適合しておらず、基本的なヘッダチェックで弾かれている可能性も考えられる。

#### 原因
- 目的関数の設定誤り
- メモリ制限が強すぎる
- 入力ファイルが不適切

#### 解決方法
- 目的関数の再考
- メモリ制限の緩和
- 入力ファイル（seed）の再検討

### 途中から実行速度がとても遅くなった
exec speedが```-t```で指定した時間の逆数になっていないか確認せよ。もしそうであれば、毎回hangが発生していると思われる。

#### 原因
入力ファイルがtime out（又はcrash）を引き起こしている。

#### 解決方法
- ```-t```の指定時間を短くする
- 速度低下が終わるのを待つ

### 気が付いたら動作が止まっていてHDDの容量がなくなっていた
対象のアプリケーションが、一時的に隠しファイル（フォルダ）としてデータを生成しているのであれば、正常終了せずにそのまま削除されていない可能性がある。

#### 原因
アプリケーションの仕様

#### 解決方法
以下の様なbatファイルを実行すると良い。

```
:top
timeout 300

for /F %%a in ('dir /AD /B /W 202203*') do rd /S /Q %%a

goto top
```

### crashesディレクトリ内のファイル名は何か？
afl_fuzz.cの3677行以降が参考になる。以下は一覧である。

- EXCEPTION_ACCESS_VIOLATION
- EXCEPTION_ILLEGAL_INSTRUCTION
- EXCEPTION_PRIV_INSTRUCTION
- EXCEPTION_INT_DIVIDE_BY_ZERO
- STATUS_HEAP_CORRUPTION
- EXCEPTION_STACK_OVERFLOW
- STATUS_STACK_BUFFER_OVERRUN
- STATUS_FATAL_APP_EXIT
- EXCEPTION_NAME_NOT_AVAILABLE

