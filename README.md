# hackmetu_operation_score
OperationScore, uç Pardus makinelerinden güvenlik/sağlık metriklerini toplayıp FastAPI sunucusuna gönderen ve her cihaz için anlık skor + risk seviyesi üreten bir sistemdir. Sunucu cihaz kayıtlarını ve skor geçmişini SQLite’ta tutar, en güncel durumu tek bir snapshot JSON olarak yayınlar ve dashboard’u WebSocket ile gerçek zamanlı günceller.
