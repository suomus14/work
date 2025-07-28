@startuml
start

:tsvファイルのパスを取得する;

:tsvファイルのサイズを取得する;
/'
:tsvファイルを開く;
if (tsvファイルを開けなかった) then (yes)
    :;
elseif (tsvファイルのサイズを取得できなかった) then (yes)
    :;
    :tsvファイルを閉じる;
else (no)
    :tsvファイルを閉じる;
endif
note right
    ・ファイルがなければ何もしないでreturn
    ・ファイル情報の取得に失敗した場合もreturn
end note
'/

if (受信ログのMIDが"639_EndJob"/"8708"である) then (yes)
    :"<START_TIME>_<RECOVERY_END_TIME>_ONGOING.tsv"を"<START_TIME>_<END_TIME>.tsv"にリネームする;
    note right
        <END_TIME>：受信したログのタイムスタンプ
        <RECOVERY_END_TIME>：リカバリ期間の終了時刻
    end note
    :tsvステータスを"未作成(TSV_STATUS_NOT_YET_CREATED)"に更新する;
    :次に書き込むtsvのファイル名を"START__TIME___MARKER_<RECOVERY_END_TIME>_ONGOING.tsv"に設定する;
    note right
        この時点で、"START__TIME___MARKER_<RECOVERY_END_TIME>_ONGOING.tsv"はまだ作成していない。
    end note
    stop
elseif (書き込むtsvのファイル名に"START__TIME___MARKER"が含まれている) then (yes)
    :次に書き込むtsvのファイル名を"START__TIME___MARKER_<RECOVERY_END_TIME>_ONGOING.tsv"を"<START_TIME>_<RECOVERY_END_TIME>_ONGOING.tsv"に設定する;
    note right
        <START_TIME>：受信したログのタイムスタンプ
        <RECOVERY_END_TIME>：リカバリ期間の終了時刻
    end note
    :tsvファイルを作成する;
    :tsvステータスを"空(TSV_STATUS_EMPTY)"に更新する;
    note right
        "639_EndJob"/"8708"の次にログを受信したらここに入る。
    end note
    stop
elseif (受信ログのMIDが"105"である) then (yes)
    :nop;
elseif (アイドル時のログ かつ tsvファイルが閾値(100MB)を超えた) then (yes)
    :nop;
else (no)
    stop
endif

:"<START_TIME>_<RECOVERY_END_TIME>_ONGOING.tsv"を"<START_TIME>_<END_TIME>.tsv"にリネームする;




















stop
@enduml
