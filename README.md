＜課業１＞
・予実・課題管理について
上半期は案件が少なく、体系的な予実管理を行う機会は限られていた。また、課題の発生も少なく、明確な実績は乏しい結果となった。今後は、小規模案件でも簡易的な予実・課題記録を積極的に行い、管理精度の向上を図る。

・緊急対応における計画遵守
市場トラブル対応においては、突発的な依頼にも関わらず、予定工数内で作業を完了し、品質と納期を両立させることができた。これは、事前準備と柔軟な対応力の成果と捉えている。

・作業効率化の取り組み
サーバーへの自動ログインツールを作成し、作業の削減を図った。現在は個人利用の段階だが、今後はチームへの展開を進め、全体の業務効率化に寄与する。



＜課業２＞
一括委託案件が少ない上半期においては、受託稼働率の維持を目的に、実績清算委託での対応を積極的に活用した。市場トラブル対応の際には、チャットを通じてリアルタイムに状況を把握し、委託元へ調査事項を提案するなど、谷間の稼働を自ら創出する動きを意識的に実践した。これにより、待ちの姿勢にとどまらず、提案型のアプローチで稼働率の安定化につなげることができた。



＜課業３＞
・手戻りへの対応
上半期は案件が少なく、主に実績清算ベースでの対応が中心となった。一部案件で仕様変更に伴う手戻りが発生したが、これは委託元からの追加仕様によるものであり、必要な対応と判断。実績清算内で対応可能であったため、工程全体への影響はなく、問題ない範囲で収めることができた。また、仕様調整段階からの関与により、変更点のスムーズな実装に貢献できた。

・技術習得とシステム理解の深化
障害対応を通して、システム構成や動作仕様の理解を深めることができた。特に、既存モジュールの挙動について実務経験を積むことで、今後の開発での技術的判断や早期問題発見につながる基礎力を養えたと考えている。



＜課業４＞
CM活動では、予定工数内での作業を維持しつつ、新入社員に対して単に答えを教えるのではなく、ヒントを与えて自ら考えて行動するよう促すフィードバックを意識的に実践した。これにより、自主性を育てるコーチングのスタイルを少しずつ確立できていると感じている。
また、安全衛生活動においては、定期的な連絡とリマインドの徹底により、課内全体での期日厳守が実現できており、活動の安定運営に寄与している。





@startuml aaa
actor Thread
participant LogCollector
participant DcdpConnector
participant L0MsgSpecMap
participant DcdpAutoRecovery
participant RepoAdapter

Thread -> LogCollector: startRoutine()
loop while !isTerminated()
    LogCollector -> DcdpConnector: isConnectedToDcdp()
    alt 未接続
        LogCollector -> DcdpConnector: connectToDcdp()
        alt 接続失敗
            LogCollector -> LogCollector: sleep(1)
            continue
        else 接続成功
            LogCollector -> DcdpConnector: getL0MsgSpecMap()
            LogCollector -> L0MsgSpecMap: getBrtlVersion()
        end
    end

    LogCollector -> DcdpConnector: resetDisconnectDetectionFlag()
    LogCollector -> DcdpConnector: getLogFromDcdp(logType, logData)
    alt err == -1 (取得失敗)
        LogCollector -> DcdpConnector: stopLogRequest()
        LogCollector -> DcdpConnector: disconnectFromDcdp()
        LogCollector -> LogCollector: sleep(1)
        alt m_autoRecoveryFlag
            LogCollector -> DcdpAutoRecovery: getRetransmissionTime()
            LogCollector -> DcdpConnector: setRetransmissionTime()
            LogCollector -> DcdpConnector: setRetransmissionMode()
        else
            LogCollector -> LogCollector: cleanUp()
        end
        continue
    else err == -2 (End of Transmission)
        alt m_autoRecoveryFlag
            LogCollector -> LogCollector: finalizeTSVFile()
            LogCollector -> DcdpAutoRecovery: notifyRecoveryEnd()
        end
        break
    else 正常取得
        LogCollector -> DcdpConnector: getDisconnectDetectionFlag()
        alt disconnect detected
            alt !m_autoRecoveryFlag
                LogCollector -> DcdpConnector: disconnectFromDcdp()
                LogCollector -> LogCollector: cleanUp()
                continue
            end
        end

        alt logType == L0
            LogCollector -> logData: getMsgId()
            LogCollector -> logData: getContext()
            LogCollector -> logData: getSec(), getMicrosec()
            LogCollector -> L0MsgSpecMap: extractParameters()
            LogCollector -> logData: validateParams()
            alt m_autoRecoveryFlag
                LogCollector -> DcdpAutoRecovery: isRecoveryEnd()
                alt EndJob/ResetEnd/MaxThreshold
                    LogCollector -> LogCollector: saveMessageToTSV()
                    LogCollector -> LogCollector: finalizeTSVFile()
                    LogCollector -> DcdpAutoRecovery: notifyRecoveryEnd()
                    break
                else
                    LogCollector -> LogCollector: saveMessageToTSV()
                end
            else
                LogCollector -> RepoAdapter: executeMessageHandler()
                alt hasAppendedData
                    LogCollector -> LogCollector: saveMessageToTSV(..., true)
                end
            end
        else logType == VERSION
            LogCollector -> DcdpConnector: setRetransmissionTime()
            LogCollector -> DcdpConnector: setRetransmissionMode()
            LogCollector -> DcdpConnector: stopLogRequest()
            LogCollector -> DcdpConnector: disconnectFromDcdp()
        end
    end
end

alt DcdpConnector::isConnectedToDcdp()
    LogCollector -> DcdpConnector: disconnectFromDcdp()
end
LogCollector -> LogCollector: m_runningFlag = false
@enduml
