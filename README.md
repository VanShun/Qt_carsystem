简易的桌面系统，包含4个子功能（音频、视频播放、天气预报、地图）用于练手
其中，音视频播放主要运用了mplayer，QProgress等；关于mplayer的使用，参考：https://www.cnblogs.com/huangpeng1990/p/4364373.html  主要是一些命令行指令
天气预报和地图主要运用了QJason及网络请求QNetworkAcessManager/QNetworkReply等相关类，调用一些api，然后解析Jason数据，下面是一段应该挺典型通用的Jason数据及其解析代码：

/*
{
"reason":"查询成功!",
"result":{"city":"厦门",
    "realtime":{"temperature":"25","humidity":"60","info":"多云","wid":"01","direct":"东北风","power":"2级","aqi":"29"},
    "future":[{"date":"2021-10-19","temperature":"22\/28℃","weather":"多云","wid":{"day":"01","night":"01"},"direct":"东风转东北风"},
        {"date":"2021-10-20","temperature":"22\/26℃","weather":"多云转阴","wid":{"day":"01","night":"02"},"direct":"东风转持续无风向"},
        {"date":"2021-10-21","temperature":"20\/24℃","weather":"小雨转阴","wid":{"day":"07","night":"02"},"direct":"东北风"},
        {"date":"2021-10-22","temperature":"19\/21℃","weather":"阴","wid":{"day":"02","night":"02"},"direct":"东北风"},
        {"date":"2021-10-23","temperature":"18\/21℃","weather":"阴转多云","wid":{"day":"02","night":"01"},"direct":"东北风转北风"}]
    },
"error_code":0
}
*/


//处理收到的 json 数据
void WeatherWindow::replyFinished(QNetworkReply *reply)
{
    QByteArray array = reply->readAll();
    QJsonParseError error;
    QJsonDocument doc = QJsonDocument::fromJson(array, &error);
    QString temperature, humidity, weather, wind, aqi, date, dir;

    if(error.error !=QJsonParseError::NoError)
    {
        qDebug("josn error");
        return ;
    }
    QJsonObject obj = doc.object();
    QString reason = obj.value("reason").toString();
    if (reason != "查询成功!") {
        qDebug() << reason << endl;
        return;
    }
    QJsonObject resultobj = obj.value("result").toObject();
    QString city = resultobj.value("city").toString();
    qDebug() << "city:" << city << endl;
    QJsonObject realtimeobj = resultobj.value("realtime").toObject();
    temperature = realtimeobj.value("temperature").toString();
    humidity = realtimeobj.value("humidity").toString();
    weather = realtimeobj.value("info").toString();
    qDebug() << QString("realtime temperature:%1, humidity:%2, weather:%3").arg(temperature).arg(humidity).arg(weather) << endl;
    ui->label_local_temperature->setText(temperature.toUtf8() + "℃");
    //将保存在本文件中的天气图标用链接显示出来
    ui->label_local_weather->setStyleSheet(QString("border-image: url(:/pic/%1);").arg(get_weather_pic(weather)));

    QJsonArray forecast_array = resultobj.value("future").toArray();
    qDebug() << forecast_array.count() << endl;
    for (int i = 0; i < forecast_array.count(); i++) {
        QJsonObject weatherobj = forecast_array.at(i).toObject();
        date = weatherobj.value("date").toString();
        temperature = weatherobj.value("temperature").toString();
        weather = weatherobj.value("weather").toString();
        QJsonObject widobj = weatherobj.value("wid").toObject();
        qDebug() << QString("date:%1, temperature:%2, weather:%3").arg(date).arg(temperature).arg(weather) << endl;
        m_vec_labeld.at(i)->setText(date.toUtf8());
        m_vec_labelt.at(i)->setText(temperature.toUtf8());
        m_vec_labelw.at(i)->setStyleSheet(QString("border-image: url(:/pic/%1);").arg(get_weather_pic(weather)));
        m_vec_labelwtxt.at(i)->setText(weather.toUtf8());
    }
}
