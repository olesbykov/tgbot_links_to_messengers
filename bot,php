<?php
/**
 * Самая простая версия бота, берёт все цифры из сообщения и считает их номером телефона
 * https://api.telegram.org/bot<token>/setWebhook?url=https://...bot's.link.../bot.php - выполнить для того, чтобы Телеграм передавал запросы к боту на скрипт
 * http://t.me/MessengerLinksBot - здесь запущен аналогичный скрипт
 */
$data = file_get_contents('php://input');//включаем возможность получить данные от Телеграм
$data = json_decode($data, true);//превращаем json запрос от Телеграм в массив

if (empty($data['message']['chat']['id'])) { //это если нет информации об отправителе. Условие сработает, если, например, обратиться к скрипту из браузера.
    http_response_code(404);
    echo "Привет, как дела?";
    exit();
}

define('TOKEN', '<token>'); // <token> - ваш токен из BotFather вида bot123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11

function phonenumber(&$phone){ //используем ссылку (&), чтобы переменную можно было использовать после обработки
    $phone = preg_replace('/\D+/', '', $phone);//заменяем все не цифры на пустоту, /\D+/ - аналог /[^0-9]/
    if($phone[0] === '8') { //считаем все номера на 8 российскими. Аналогично под другие страны: проверяем, если 0, то его убираем и ставим код нужной страны
        $phone = mb_substr($phone, 1 );//удаляем один символ
        $phone = '7' . $phone; //добавим код страны вместо удалённой восьмёрки
    }
}

function sendTelegram($method, $response) //функция отправки сообщений в Телеграм
{
    $ch = curl_init('https://api.telegram.org/bot' . TOKEN . '/' . $method);
    curl_setopt($ch, CURLOPT_POST, 1);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $response);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HEADER, false);
    $res = curl_exec($ch);
    curl_close($ch);

    return $res;
}

if (!empty($data['message']['text'])) { //если есть текст, то отвечаем на него
    $text = $data['message']['text']; //сначала запишем его в переменную

    if ($text === "/start") { // если пользователь только запустил бота
        $reply = "<b>Добро пожаловать!</b>
Этот бот поможет сгенерировать ссылки на мессенджеры.

<b>Внимание:</b> номера на 8 распознаются как российские."; //переносы сохраняются, тег <br/> мешает отправить сообщения, поэтому не используйте его
        sendTelegram(//вызываем функцию отправки в Телеграм. Надо бы ещё упростить, чтобы на этом этапе подставлять только $reply
            'sendMessage',
            array(
                'chat_id' => $data['message']['chat']['id'],
                'parse_mode' => 'html',
                'text' => $reply
            )
        );
    } elseif ($text === "/help") {
        $reply = "Введите номер телефона с кодом страны. Этот бот превратит телефонные номера в ссылки на мессенджеры.

Также бот генерирует три ссылки – на Telegram, Viber и WhatsApp.

Надеюсь, что @MessengerLinksBot сделает ваше общение приятнее.";
        sendTelegram(
            'sendMessage',
            array(
                'chat_id' => $data['message']['chat']['id'],
                'parse_mode' => 'html',
                'text' => $reply )
        );
    } else {//условие для всех текстов, что не заданные команды /start и /help
        $phone = $text;//это для читаемости кода, а то превращать $text в цифры в функции как-то странно. Прям мешает.
        phonenumber($phone);//отправляем нашу строку в функцию
        if ($phone === '') {//если в строке нет ни одной цифры
            $resultMessage = "Телефон не распознан";
        } elseif (strlen($phone)<10) {//если цифры есть, но их меньше 10
            $resultMessage = "Телефон подозрительно короткий";
        } else {//если цифр больше 10, то, возможно, это телефон. Собираем ссылки
            $resultMessage = "Распознан телефон <b>+" . $phone . "</b>

<a href=\"https://t.me/+" . $phone . "\">Перейти в Телеграм</a>


Telegram: <code>https://t.me/+" . $phone . "</code>


<a href=\"https://wa.me/" . $phone . "\">Перейти в WA</a>


WhatsApp: <code>https://wa.me/" . $phone . "</code>


Viber: <code>viber://chat?number=%2B" . $phone . "</code>";
        }
        //<a href=\"viber://chat?number=%2B".$phone."\">Перейти в Viber</a> - так ссылка не работает, а домена у Viber нет

        sendTelegram(
            'sendMessage',
            array(
                'chat_id' => $data['message']['chat']['id'],
                'parse_mode' => 'html',
                'disable_web_page_preview' => true,
                'text' => $resultMessage
            )
        );

        exit();
    }
}
/*
 *
                'chat_id' => $data['message']['chat']['id'], //это чат айди - куда отправить боту
                'parse_mode' => 'html', //это разрешение на использование html-тегов <b>, <a>, <i>, <code>, <pre>
                'disable_web_page_preview' => true, //это запрещает создавать превью ссылкам
                'text' => $resultMessage //отправляем $resultMessage
 */
