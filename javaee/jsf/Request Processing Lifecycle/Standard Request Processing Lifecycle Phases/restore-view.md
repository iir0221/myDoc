# Restore View

* postback是什么意思
* req是postback，所做的处理
    * setProcessingEvents
* req不是postback，所做的处理
    * 什么情况下，直接render：
        * vld！== null & metadata！== null & !ViewMetadata.hasMetadata(viewRoot)
        * null == vdl || null == metadata
    

