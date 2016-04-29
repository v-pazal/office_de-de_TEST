
# Überprüfen eines Identitätstokens mit PHP
In diesem Thema erfahren Sie, wie Sie mit PHP das Exchange 2013-Identitätstoken überprüfen, das die E-Mail-Konten Ihres Outlook-Add-In-Kunden mit den von Ihrem Dienst bereitgestellten Informationen verknüpft.

 _**Gilt für:** apps for Office | Office Add-ins | Outlook_

Ihr Outlook-Add-In kann Ihnen ein Identitätstoken senden. Bevor Sie der Anforderung vertrauen, müssen Sie jedoch das Token überprüfen, um sicherzustellen, dass es auch wirklich von dem Exchange-Server stammt. In dem Beispiel in diesem Artikel wird PHP verwendet, um das Identitätstoken zu überprüfen. Für die Überprüfung können Sie jedoch eine beliebige Programmiersprache verwenden. Die zum Überprüfen des Tokens erforderlichen Schritte sind im [JSON Web Token (JWT)-Internetentwurf](http://self-issued.info/docs/draft-goland-json-web-token-00.mdl) beschrieben.

Es wird empfohlen, einen aus vier Schritten bestehenden Vorgang zum Überprüfen des Identitätstokens und Abrufen der eindeutigen ID des Benutzers zu verwenden. 


1. Extrahieren Sie das JWT aus einer base64-codierten Zeichenfolge.
    
2. Stellen Sie sicher, dass das Token wohlgeformt ist, dass es für Ihr Outlook-Add-in bestimmt und nicht abgelaufen ist und dass Sie eine gültige URL für das Dokument mit den Authentifizierungsmetadaten extrahieren können.
    
3. Rufen Sie das Dokument mit den Authentifizierungsmetadaten vom Exchange-Server ab und überprüfen Sie die an das Token angefügte Signatur.
    
4. Berechnen Sie eine eindeutige ID für den Benutzer, indem Sie einen Hash für die Exchange-ID des Benutzers mit der URL des Authentifizierungsmetadaten-Dokuments erstellen.
    
Dieser Vorgang scheint insgesamt komplex zu sein, die einzelnen Schritte sind jedoch ziemlich einfach. Sie können sehen, dass die PHP-Bibliothek den Prozess in genau dieser Weise enthält, wenn Sie den Code der Prüffunktion betrachten.



```
   static function validate($token, $phpseclib_path, $audiences,
      &amp;$user_id=NULL, &amp;$user_email=NULL, $debug=FALSE)
   {
      $valid_token = NULL;
      $user_id = FALSE;

      if(!empty($token) AND is_string($token))
      {
         // Extract the JWT from a base64 URL-encoded string.
         self::parse_token($token, $valid_token, $header, $payload, $signature, $jws_secured_input, $debug);

         // Make sure that the token is well-formed, is for your Outlook add-in,
         // that it has not expired, and that you can extract a valid URL 
         // for the authentication metadata document.
         self::validate_header($header, $valid_token, $x5t, $debug);

         // Retrieve the authentication metadata document from the Exchange server
         // and validate the signature attached to the token.
         self::validate_payload($payload, $phpseclib_path, $audiences, $x5t,
            $jws_secured_input, $signature, $valid_token, $appctx_ar, $debug);

         // Compute a unique identifier for the user.
         if($valid_token)
         {
            $user_id = self::get_user_id($appctx_ar, $user_email, $debug);
         }
      }
      else
      {
         if($debug) echo 'invalid token' . "\n";
         $valid_token = FALSE;
      }

      if($valid_token !== TRUE) $valid_token = FALSE;

      if($debug) echo '$valid_token: ' . $valid_token . "\n";

      return $valid_token;
   }

```


## Überprüfungsbibliothek für Identitätstoken

Der folgende Code ist eine PHP-Bibliothek zum Überprüfen eines Exchange-Identitätstokens.


```
<?php
/*
Exchange Identity Token Validator for Office Add-ins
A PHP library

Depends on phpseclib v0.2.2 - found at: http://phpseclib.sourceforge.net/

Created by Scott Otis - CTO &amp; Co-Founder - Intand Corporation
Website: www.tandemcal.com

Additional help provided by:
Andrew Salamatov - Microsoft Corporation

Microsoft licenses this code sample to you under the terms of the Microsoft Limited Public License, 
   (http://msdn.microsoft.com/en-us/cc300389.aspx#O).

Documentation:

token_validator::validate($token, $phpseclib_path, $audiences, &amp;$user_id=NULL,
   &amp;$user_email=NULL, $debug=FALSE)

Parameters:
   $token (string): The Exchange identity token.

   $phpseclib_path (string): The path to the location of the phpseclib. This path will be added to
                             the include_path configuration option using the set_include_path() function.

   $audiences (array): The URL(s) from the SourceLocation element in your Add-in for Office XML Manifest.

   $user_id (passed by reference): If the token is valid, this will be set on return to an MD5 hash
                                   that uniquely identifies the user. If the token is not valid or
                                   if there is an issue generating the unique ID for the user, this
                                   will be set on return to an MD5 hash of the concatination of the
                                   amurl and msexchuid values from the payload's appctx property.

   $user_email (passed by reference): If the token is valid, this will be set on return to the
                                      SMTP email address of the user. If the token is not valid,
                                      or if there is an issue generating the email address, it
                                      will be set to FALSE.

   $debug (boolean): True to echo debug information, otherwise, false to suppress debug information.

Return:
   boolean TRUE/FALSE

Example Usage:
   $token = $_REQUEST['token'];
   $phpseclib_path = 'includes/phpseclib';

   $audiences[] = 'http://www.example.com/officeapp/index.html';
   $audiences[] = 'https://www.example.com/officeapp/index.html';

   $user_id = NULL;
   $user_email = NULL;

   $debug = FALSE;

   $token_valid = token_validator::validate($token, $phpseclib_path, $audiences,
      $user_id, $user_email, $debug);
*/

class token_validator
{
   static function validate($token, $phpseclib_path, $audiences,
      &amp;$user_id=NULL, &amp;$user_email=NULL, $debug=FALSE)
   {
      $valid_token = NULL;
      $user_id = FALSE;

      if(!empty($token) AND is_string($token))
      {
         self::parse_token($token, $valid_token, $header, $payload, $signature, $jws_secured_input, $debug);

         self::validate_header($header, $valid_token, $x5t, $debug);

         self::validate_payload($payload, $phpseclib_path, $audiences, $x5t,
            $jws_secured_input, $signature, $valid_token, $appctx_ar, $debug);

         if($valid_token)
         {
            $user_id = self::get_user_id($appctx_ar, $user_email, $debug);
         }
      }
      else
      {
         if($debug) echo 'invalid token' . "\n";
         $valid_token = FALSE;
      }

      if($valid_token !== TRUE) $valid_token = FALSE;

      if($debug) echo '$valid_token: ' . $valid_token . "\n";

      return $valid_token;
   }

   static function parse_token($token, &amp;$valid_token, &amp;$header, &amp;$payload,
      &amp;$signature, &amp;$jws_secured_input, $debug=FALSE)
   {
      $token_ar = explode('.', $token);

      if($debug)
      {
         echo '$token_ar: ' . print_r($token_ar, TRUE) . "\n";
      }

      if(count($token_ar) == 3)
      {
         $encoded_header = $token_ar[0];
         $encoded_payload = $token_ar[1];
         $encoded_signature = $token_ar[2];

         $jws_secured_input = $encoded_header . '.' . $encoded_payload;

         $header = json_decode( self::rfc4648_base64_url_decode($encoded_header), TRUE);
         $payload = json_decode( self::rfc4648_base64_url_decode($encoded_payload), TRUE);
         $signature = self::rfc4648_base64_url_decode($encoded_signature);

         if($debug)
         {
            echo '$header: ' . print_r($header, TRUE) . "\n";
            echo '$payload: ' . print_r($payload, TRUE) . "\n";
            echo '$signature: ' . $signature . "\n";

            echo '$jws_secured_input: ' . $jws_secured_input . "\n";
          }
      }
      else
      {
         if($debug) echo 'invalid token' . "\n";
         $valid_token = FALSE;
      }
   }

   static function rfc4648_base64_url_decode($url)
   {
      $url = str_replace('-', '+', $url); // 62nd char of encoding
      $url = str_replace('_', '/', $url); // 63rd char of encoding

      switch (strlen($url) % 4) // Pad with trailing '='s
      {
         case 0:
            // No pad chars in this case
            break;
         case 2:
            // Two pad chars
            $url .= "==";
            break;
         case 3:
            // One pad char
            $url .= "=";
            break;
         default:
            $url = FALSE;
      }

      if($url) $url = base64_decode($url);

      return $url;
   }

   static function validate_header($header, &amp;$valid_token, &amp;$x5t, $debug=FALSE)
   {
      if(!empty($header) AND is_array($header))
      {
         if($header['typ'] != 'JWT')
         {
            if($debug) echo 'bad header type' . "\n";

            $valid_token = FALSE;
         }

         if($header['alg'] != 'RS256')
         {
            if($debug) echo 'bad header alg' . "\n";

            $valid_token = FALSE;
         }

         $x5t = $header['x5t'];
      }
      else
      {
         if($debug) echo 'invalid header' . "\n";

         $valid_token = FALSE;
      }
   }

   static function validate_payload($payload, $phpseclib_path, $audiences, $x5t, $jws_secured_input, 
      $signature, &amp;$valid_token, &amp;$appctx_ar, $debug=FALSE)
   {
      set_include_path(get_include_path() . PATH_SEPARATOR . $phpseclib_path);

      require_once('Crypt/RSA.php');

      if(!empty($payload) AND is_array($payload))
      {
         $prev_tz = date_default_timezone_get();

         //if($debug) echo '$prev_tz: ' . $prev_tz . "\n";

         date_default_timezone_set('UTC');

         $now = time();

         if($now <= $payload['nbf'] OR $now >= $payload['exp'])
         {
            if($debug)
            {
               echo 'bad payload nbf / exp' . "\n";

               echo '$now: ' . date('Y-m-d H:i:s', $now) . " UTC\n";
               echo 'nbf: ' . date('Y-m-d H:i:s', $payload['nbf']) . " UTC\n";
               echo 'exp: ' . date('Y-m-d H:i:s', $payload['exp']) . " UTC\n";
            }

            $valid_token = FALSE;
         }

         if(!in_array($payload['aud'], $audiences))
         {
            if($debug) echo 'bad payload aud' . "\n";

            $valid_token = FALSE;
         }

         if(!empty($payload['appctx']))
         {
            $appctx_ar = json_decode($payload['appctx'], TRUE);

            if($debug) echo '$appctx_ar: ' . print_r($appctx_ar, TRUE) . "\n";

            $amurl = $appctx_ar['amurl'];

            self::validate_amurl($amurl, $x5t, $jws_secured_input, $signature, $valid_token, $debug);
         }
         else
         {
            if($debug) echo 'empty payload appctx' . "\n";

            $valid_token = FALSE;
         }

         date_default_timezone_set( $prev_tz );
      }
      else
      {
         if($debug) echo 'invalid payload' . "\n";

         $valid_token = FALSE;
      }
   }

   static function validate_amurl($amurl, $x5t, $jws_secured_input, $signature, &amp;$valid_token, $debug=FALSE)
   {
      if(!empty($amurl))
      {
         if($debug) echo '$amurl: ' . $amurl . "\n";

         $auth_metadata = file_get_contents($amurl);

         if($auth_metadata)
         {
            $auth_metadata = json_decode($auth_metadata, TRUE);

            if($debug) echo '$auth_metadata: ' . print_r($auth_metadata, TRUE) . "\n";

            if(!empty($auth_metadata['keys']) AND is_array($auth_metadata['keys']))
            {
               $good_key = FALSE;

               foreach($auth_metadata['keys'] as $key => $value)
               {
                  $good_key = self::is_good_key($value, $x5t);

                 if($good_key)
                  {
                     $x509_public_key = self::get_x509_public_key($value['keyvalue']['value'], $debug);

                     $validated = self::validate_token($x509_public_key, $jws_secured_input, $signature);

                     if($validated === TRUE)
                     {
                        if($valid_token !== FALSE) $valid_token = TRUE;
                     }
                     else
                     {
                        $valid_token = FALSE;

                        if($debug) echo 'invalid token' . "\n";
                     }
    
                     break;
                  }
               }

               if(!$good_key)
               {
                  if($debug) echo 'did not find valid auth metadata key' . "\n";

                  $valid_token = FALSE;
               }
            }
            else
            {
               if($debug) echo 'invalid auth metadata keys' . "\n";

               $valid_token = FALSE;
            }
         }
         else
         {
            if($debug) echo 'invalid auth metadata' . "\n";

            $valid_token = FALSE;
         }
      }
      else
      {
         if($debug) echo 'empty payload appctx amurl' . "\n";

         $valid_token = FALSE;
      }
   }

   static function is_good_key($value, $x5t)
   {
      $good_key = FALSE;

      if(!empty($value['keyinfo']['x5t']) AND !empty($value['keyvalue']['value']))
      {
         if($value['keyinfo']['x5t'] == $x5t)
         {
            $good_key = TRUE;

            if($debug) echo 'good_key: ' . $key . ':' . print_r($value, TRUE) . "\n";
         }
      }

      return $good_key;
   }

   static function get_x509_public_key($x509_raw, $debug=FALSE)
   {
      $x509_raw_chunked = chunk_split($x509_raw, 64);

      $x509_text = '-----BEGIN CERTIFICATE-----' . "\n" . $x509_raw_chunked . '-----END CERTIFICATE-----';

      //$x509_ar = openssl_x509_parse($x509_text);

      $x509_obj = openssl_x509_read($x509_text);

      $x509_public_key_obj = openssl_pkey_get_public($x509_obj);

      $x509_public_key_ar = openssl_pkey_get_details($x509_public_key_obj);

      if($debug) echo '$x509_public_key_ar: ' . print_r($x509_public_key_ar, TRUE) . "\n";

      $key = $x509_public_key_ar['key'];

      $key_ar = explode("\n", $key);

      $key_line_count = count($key_ar);

      unset($key_ar[0]);
      unset($key_ar[ $key_line_count - 1 ]);
      unset($key_ar[ $key_line_count - 2 ]);

      $key2 = implode('', $key_ar);

      if($debug) echo '$x509_public_key: ' . $key2 . "\n";

      return $key2;
   }

   static function validate_token($x509_public_key, $jws_secured_input, $signature)
   {
      $rsa = new Crypt_RSA();
      $rsa->setHash('sha256');
      $rsa->setMGFHash('sha256');

      $rsa->setSignatureMode( CRYPT_RSA_SIGNATURE_PKCS1 );

      $rsa->loadKey( $x509_public_key );

      $verified = $rsa->verify($jws_secured_input, $signature);

      return $verified;
   }

   static function get_user_id($appctx_ar, &amp;$smtp=NULL, $debug=FALSE)
   {
      $return = FALSE;

      if(!empty($appctx_ar) AND is_array($appctx_ar))
      {
         $amurl = $appctx_ar['amurl'];
         $smtp = $appctx_ar['smtp'];
         $msexchuid = $appctx_ar['msexchuid'];

         /*if($debug)
         {
            echo 'payload appctx amurl: ' . $amurl . "\n";
            echo 'payload appctx smtp: ' . $smtp . "\n";
            echo 'payload appctx msexchuid: ' . $msexchuid . "\n";
         }*/

         if(!empty($msexchuid))
                        {
            $return = md5($amurl . $msexchuid);
         }
         else
         {
            if($debug) echo 'empty payload appctx msexchuid' . "\n";
         }
      }
      else
      {
         if($debug) echo 'empty payload appctx' . "\n";
      }

      return $return;
   }
}

?>
```


## Zusätzliche Ressourcen



- [Authentifizieren eines Outlook-Add-Ins mithilfe von Exchange-Identitätstoken](c0520a1e-d9ba-495a-a99f-6816d7d2a23e.md)
    
- [Überprüfen eines Exchange-Identitätstokens](8503a3e8-458a-4a4e-9e95-65cd7bb1954d.md)
    
- [Innenleben des Exchange-Identitätstokens](e4085245-73ae-4480-98ac-15593daa5fd7.md)
    