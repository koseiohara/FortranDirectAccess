Programmer : Kosei Ohara
Last Update : 2024/04/26

概要 :
    Fortran で直接探査型のバイナリファイルの読み書きをする module.
    /mnt/hail1/Tools/ 内にある NaNchecker.f90 が必要ですが、use-statement と isNaN() を呼ぶ if-block を
    コメントアウトすれば独立して動作します。
    単精度実数と倍精度実数のスカラー、一次元配列、二次元配列、三次元配列に対応しており、実引数の型に対して
    コンパイラが適切な関数を選択します。


構造体 :
    専用の構造体 finfo を用いてファイル操作をします
    これを使わずに module 内の関数を使用できない点に注意

    finfo
        unit :
            integer
            4byte
            ファイルの装置番号

        fname :
            character
            読み書きに使用するファイル名
            最大文字数は128に設定されているが、拡張してもよい

        action :
            character
            ファイルに対する操作
            write/read/readwrite
            最大文字数は16

        recl :
            integer
            4byte
            open-statement で指定するrecl

        record :
            integer
            4byte
            アクセスするレコード

        recstep :
            integer
            4byte
            読み書きの1ステップごとに更新するレコードの増加数


関数 :
    構造体に対する操作とファイルの読み書きは、この module 内で定義された関数を用いて行うことを想定している
    思いつく限りの安全装置をかけているため、これらの使用を推奨する

    fopen
        subroutine
        構造体に対して引数で指定した値を代入し、ファイルを開く
        open の失敗は、open() の iostat 引数によって判定され、ERROR STOP がかかる
        open の形式は、FORM='UNFORMATTED', ACCESS='DIRECT' のみを想定する
        open() に与える実引数は次から成る :
            UNIT/NEWUNIT
            FILE
            ACTION
            FORM
            ACCESS
            RECL
            IOSTAT
        その他を用いる場合は、各自で書き換えること

        以下に引数の説明をする

        ftype :
            finfo型の構造体
            intent(out)
            構造体の中身はその他の引数を用いて設定するため、空のfinfo型を実引数とする

        unit :
            integer
            4byte
            intent(in)
            optional
            ファイルの装置番号
            指定した場合、その値を装置番号として open する
            このとき、5または6を指定した場合は ERROR STOP する
            省略した場合、open 時に newunit で自動的に設定される

        fname :
            character
            intent(in)
            開くファイルの名前

        action :
            character
            intent(in)
            ファイルに対する操作
            write/read/readwrite

        recl :
            integer
            4byte
            intent(in)
            1レコードあたりのデータサイズ
            0以下を指定した場合は ERROR STOP する

        record :
            integer
            4byte
            intent(in)
            ファイルに対する操作の第1ステップで read()/write() に与えるレコード
            0以下を指定した場合は ERROR STOP する

        recstep :
            integer
            4byte
            intent(in)
            ファイルに対する操作の1ステップごとの record の増加量
            fread()/fwrite() でレコードが recstep ごと増加する
            各ステップで手動でレコードをしちしたい場合は0とする
            0未満を指定した場合は ERROR STOP する

    fclose
        subroutine
        ファイルを閉じ、構造体の値をエラー状態に書き換える
        すでに閉じられたファイルに対してこの関数を使用した場合は ERROR STOP する

        ftype
            finfo型の構造体
            intent(inout)
            この構造体が指定するファイルを閉じ、unit, recl, record, recstep を0 に、fname, action を ERROR に書き換える

    fread
        subroutine
        指定したファイルからデータを読み込む
        閉じられた/開かれていないファイルを指定した場合は ERROR STOP する
        読み込みのために定義された8つの関数の総称名として定義されている
        読み込みにNaNが含まれていた場合、警告を発する(ERROR STOP しない)

        ftype
            finfo型の構造体
            intent(inout)
            この構造体が持つ unit と record の情報をもとにデータを読み込む
            読み込んだのち、record に対して recstep を加える

        carrier
            real
            4byte or 8byte
            intent(out)
            スカラーと一次元配列、二次元配列、三次元配列に対応する
            実引数の型に対してコンパイラが8つの関数から適したものを選択する
            読み込んだデータを返す

    fwrite
        subroutine
        指定したファイルにデータを出力する
        閉じられた/開かれていないファイルを指定した場合は ERROR STOP する
        書き出しのために定義された8つの関数の総称名として定義されている
        出力データにNaNが含まれていた場合、警告を発する(ERROR STOP しない)

        ftype
            finfo型の構造体
            intent(inout)
            この構造体が持つ unit と record の情報をもとにデータを書き出す
            書き出したのち、record に対して recstep を加える

        carrier
            real
            4byte or 8byte
            intent(in)
            スカラーと一次元配列、二次元配列、三次元配列に対応する
            実引数の型に対してコンパイラが8つの関数から適したものを選択する
            持っているデータを書き出す

    reset_record
        subroutine
        finfo型構造体の持つレコードの値を書き換える
        newrecord, recstep は optional だが、両方を省略した場合は ERROR STOP する
        両方を指定しなかった場合、newrecord のみが使用される

        ftype
            finfo型の構造体
            intent(inout)
            その他の引数をもとにレコード値のみが書き換えられる

        newrecord
            integer
            intent(in)
            optional
            新しいレコード値としてrecordに代入される

        recstep
            integer
            intent(in)
            optional
            現在のレコードに対してこの値を加算する
            
    get_record
        function
        finfo型構造体の持つレコードの値を取得する
        finfo型構造体を直接操作しないために作成したが、この関数を使用せずにレコード値を見てもよい

        ftype
            finfo型の構造体

        result
            ftypeの持つレコード値








