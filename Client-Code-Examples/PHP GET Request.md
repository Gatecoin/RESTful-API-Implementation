GET request 
API: /Balance/Balances

// API settings
    $key = '';                // public key
    $secret = '';            // Private Key

    $uri='https://www.gatecoin.com/api/Balance/Balances/';

    $nonce=microtime(true);
    $contentType = '';
    $message = 'GET' . $uri . $contentType . $nonce;
    $hash = hash_hmac('sha256', strtolower($message), $secret,true);
    $hashInBase64 = base64_encode($hash);

    $ch = curl_init($uri);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, array('API_REQUEST_SIGNATURE:' . $hashInBase64, 'API_REQUEST_DATE:' . $nonce, 'API_PUBLIC_KEY:' . $key));

    $res = curl_exec($ch);
    print($res);
