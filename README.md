#tigerbook API doc

<ul>
    <li>GET /api/v1/getkey</li>
    <ul>
        <li>CAS protected -- should be accessed in a browser</li>
        <li>Returns a 32-character key that can be used to access the rest of the API</li>
        <li>Ex: api/v1/getkey --> '586c55266c456c676d54366c3137322d'</li>
    </ul>
    <li>GET /api/v1/undergraduates</li>
    <ul>
        <li>WSSE protected</li>
        <li>Returns a JSON list of dictionaries, with each dictionary representing a student.</li>
        <li>Dictionary fields: [first_name, last_name, full_name, class_year, net_id, res_college, hometown, home_lat, home_lng, dorm_number, dorm_building, dorm_lat, dorm_lng, major_type, major_raw, major_code, photo_link, phone_raw, mailbox, organization_raw, athletics_raw, athletics_url_raw, gender*, puid*, alias, email]
        </li>
        <li>* Must have privileged api access</li>
   </ul>
   <li>GET /api/v1/undergraduates/{netid}</li>
   <ul>
       <li>WSSE protected</li>
       <li>Returns a JSON dictionary representing the queried student</li>
       <li>Dictionary fields: [first_name, last_name, full_name, class_year, net_id, res_college, hometown, home_lat, home_lng, dorm_number, dorm_building, dorm_lat, dorm_lng, major_type, major_raw, major_code, photo_link, phone_raw, mailbox, organization_raw, athletics_raw, athletics_url_raw, gender*, puid*, alias, email]
        </li>
        <li>* Must have privileged api access</li>
   </ul>
   <li>GET /api/v1/roommates</li>
   <ul>
       <li>WSSE protected</li>
       <li>GET arguments: [dorm_building, dorm_number]</li>
       <li>Returns a list of all netids living in that room</li>
   </ul>
   <li>GET /api/v1/roommates/{netid}</li>
   <ul>
       <li>WSSE protected</li>
       <li>Returns a list of all netids of roommates of {netid}</li>
   </ul>
</ul>

###WSSE Explanation and Guide
<p>Each request to an API endpoint must include an HTTP header X-WSSE which
contains all authentication information for the request. The header includes four
important components:</p>
<ul>
<li>Date and time in W3DTF UTC format</li>
<li>Username of the API account to authenticate with</li>
<li>Nonce (random base64 string, recommended 32 characters in length)</li>
<li>Password digest</li>
</ul>
<p>To create the password digest value, the user must combine the raw nonce
value, the date, and the shared secret (in that order). The combined value
should be SHA256 hashed, and then base64 encoded.</p>
<p>Before including the nonce in the header, it should also be base64 encoded.
Here's a sample X-WSSE header value for the API account "jdoe" with shared
secret "secret": </p>
    UsernameToken Username="jdoe",PasswordDigest="b7dc8461b7a2fb5272028454d9745457", Nonce="WJ/uazuj9QSO9uTvqhbGyO4NvXriwbD3", Created="2016-07-15T14:50:17Z";
<br>
<p>Example WSSE code in Python</p>

    import hashlib
    import random
    from base64 import b64encode
    from datetime import datetime

    url = 'http://localhost:8000/api/v1/undergraduates'
    created = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ')
    nonce = ''.join([random.choice('0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ+/=') for i in range(32)])
    username = 'jdoe'
    password = 'b7dc8461b7a2fb5272028454d9745457'
    generated_digest = b64encode(hashlib.sha256(nonce + created + password).digest())
    headers = {
        'Authorization': 'WSSE profile="UsernameToken"',
        'X-WSSE': 'UsernameToken Username="%s", PasswordDigest="%s", Nonce="%s", Created="%s"' % (username, generated_digest, nonce, created)
    }

####Email alibresco@princeton.edu to request API key access