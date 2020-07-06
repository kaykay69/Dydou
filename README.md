
var querystring = require('querystring');
var crypto      = require('crypto');
var request     = require('request');

var BASE_URL = 'https://feelinsonice-hrd.appspot.com';
var STATIC_TOKEN = 'm198sOkJEn37DjqZ32lpRu76xmw288xSQ9';

// Client class
function Snapchat(user, pass) {
  if (!(this instanceof Snapchat))
    return new Snapchat();

  var auth_token;
  var conversations;
  var username = user;
  var password = pass;

  _request('/loq/login', {
    username: user,
    password: pass
  }, null, function(err, response, body) {
    /* TODO error */

    var b = JSON.parse(body);
    auth_token = b['updates_response']['auth_token'];
    conversations = b['conversations_response']; 
  });

}

function _request(path, payload, token, cb) {
  payload.timestamp = Date.now();
  payload.req_token = gen_request_token((token != (null || undefined) ? token : STATIC_TOKEN), payload.timestamp);

  var options = {
    url: BASE_URL + path,
    proxy: 'http://127.0.0.1:8888', // for charles to sniff
    qs: payload,
    method: 'POST',
    rejectUnauthorized: false,
    agentOptions: {
    },
    headers: {
      'User-Agent': 'Snapchat/8.1.1 (iPhone5,1; iOS 8.1.3; gzip)',
      'Accept-Language': 'en-US;q=1, en;q=0.9',
      'Accept-Locale': 'en',
      'Content-Type': 'application/x-www-form-urlencoded',
    }
  };

  request.post(options, cb);
}

function gen_request_token(auth_token, timestamp) {
  var pattern = '0001110111101110001111010101111011010001001110011000110001000110';
  var secret = 'iEk21fuwZApXlz93750dmW22pw389dPwOk';

  var first = crypto.createHash('sha256').update(secret + auth_token, 'ascii').digest('hex');
  var second = crypto.createHash('sha256').update(timestamp + secret, 'ascii').digest('hex');

  var bits = '';
  for (i in pattern) {
    (pattern[i] === '0' ? bits += first[i] : bits +=second[i]);
  }
  return bits;
}


Snapchat.prototype.something = function() {
};

module.exports = Snapchat;
