# password-cracker
password cracker online web tool
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Password Cracker Tool</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f4f6fb;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
        }
        .container {
            background: #fff;
            margin-top: 40px;
            padding: 32px 24px 24px 24px;
            border-radius: 12px;
            box-shadow: 0 4px 24px rgba(0,0,0,0.08);
            width: 100%;
            max-width: 450px;
        }
        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 24px;
        }
        label {
            display: block;
            margin-bottom: 6px;
            color: #444;
            font-weight: 500;
        }
        input[type="text"], select, textarea {
            width: 100%;
            padding: 10px;
            margin-bottom: 16px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 16px;
            box-sizing: border-box;
        }
        textarea {
            resize: vertical;
            min-height: 60px;
            max-height: 200px;
        }
        .file-upload {
            margin-bottom: 16px;
        }
        button {
            padding: 10px 18px;
            background: #4f8cff;
            color: #fff;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            cursor: pointer;
            transition: background 0.2s;
            width: 100%;
        }
        button:hover {
            background: #2563eb;
        }
        .progress {
            margin: 16px 0 8px 0;
            color: #666;
            font-size: 15px;
        }
        .result {
            margin-top: 12px;
            padding: 12px;
            border-radius: 6px;
            background: #f7f9fc;
            color: #222;
            font-size: 16px;
            word-break: break-all;
        }
        .success {
            background: #d1fae5;
            color: #15803d;
        }
        .fail {
            background: #fee2e2;
            color: #b91c1c;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Password Cracker</h1>
        <form id="cracker-form" autocomplete="off">
            <label for="hash">Hashed Password</label>
            <input type="text" id="hash" required placeholder="Enter hash (e.g., 5f4dcc3b5aa765d61d8327deb882cf99)">

            <label for="hash-type">Hash Type</label>
            <select id="hash-type" required>
                <option value="md5">MD5</option>
                <option value="sha1">SHA-1</option>
                <option value="sha256">SHA-256</option>
            </select>

            <label for="wordlist">Wordlist (one word per line)</label>
            <textarea id="wordlist" placeholder="password\n123456\nqwerty"></textarea>

            <div class="file-upload">
                <label for="wordlist-file">Or upload wordlist file (.txt)</label>
                <input type="file" id="wordlist-file" accept=".txt">
            </div>

            <button type="submit">Crack Password</button>
        </form>
        <div class="progress" id="progress"></div>
        <div class="result" id="result"></div>
    </div>
    <script>
        // Hashing functions using Web Crypto API
        async function hashString(str, type) {
            const encoder = new TextEncoder();
            const data = encoder.encode(str);
            let algo = type;
            if (type === 'md5') {
                // MD5 is not supported by Web Crypto API, use a JS implementation
                return md5(str);
            }
            if (type === 'sha1') algo = 'SHA-1';
            if (type === 'sha256') algo = 'SHA-256';
            const hashBuffer = await crypto.subtle.digest(algo, data);
            return Array.from(new Uint8Array(hashBuffer)).map(b => b.toString(16).padStart(2, '0')).join('');
        }

        // Minimal MD5 implementation (https://github.com/blueimp/JavaScript-MD5, MIT License)
        function md5cycle(x, k) {
            let a = x[0], b = x[1], c = x[2], d = x[3];
            function cmn(q, a, b, x, s, t) {
                a = (((a + q) | 0) + ((x + t) | 0)) | 0;
                return (((a << s) | (a >>> (32 - s))) + b) | 0;
            }
            function ff(a, b, c, d, x, s, t) {
                return cmn((b & c) | (~b & d), a, b, x, s, t);
            }
            function gg(a, b, c, d, x, s, t) {
                return cmn((b & d) | (c & ~d), a, b, x, s, t);
            }
            function hh(a, b, c, d, x, s, t) {
                return cmn(b ^ c ^ d, a, b, x, s, t);
            }
            function ii(a, b, c, d, x, s, t) {
                return cmn(c ^ (b | ~d), a, b, x, s, t);
            }
            a = ff(a, b, c, d, k[0], 7, -680876936);
            d = ff(d, a, b, c, k[1], 12, -389564586);
            c = ff(c, d, a, b, k[2], 17, 606105819);
            b = ff(b, c, d, a, k[3], 22, -1044525330);
            a = ff(a, b, c, d, k[4], 7, -176418897);
            d = ff(d, a, b, c, k[5], 12, 1200080426);
            c = ff(c, d, a, b, k[6], 17, -1473231341);
            b = ff(b, c, d, a, k[7], 22, -45705983);
            a = ff(a, b, c, d, k[8], 7, 1770035416);
            d = ff(d, a, b, c, k[9], 12, -1958414417);
            c = ff(c, d, a, b, k[10], 17, -42063);
            b = ff(b, c, d, a, k[11], 22, -1990404162);
            a = ff(a, b, c, d, k[12], 7, 1804603682);
            d = ff(d, a, b, c, k[13], 12, -40341101);
            c = ff(c, d, a, b, k[14], 17, -1502002290);
            b = ff(b, c, d, a, k[15], 22, 1236535329);
            a = gg(a, b, c, d, k[1], 5, -165796510);
            d = gg(d, a, b, c, k[6], 9, -1069501632);
            c = gg(c, d, a, b, k[11], 14, 643717713);
            b = gg(b, c, d, a, k[0], 20, -373897302);
            a = gg(a, b, c, d, k[5], 5, -701558691);
            d = gg(d, a, b, c, k[10], 9, 38016083);
            c = gg(c, d, a, b, k[15], 14, -660478335);
            b = gg(b, c, d, a, k[4], 20, -405537848);
            a = gg(a, b, c, d, k[9], 5, 568446438);
            d = gg(d, a, b, c, k[14], 9, -1019803690);
            c = gg(c, d, a, b, k[3], 14, -187363961);
            b = gg(b, c, d, a, k[8], 20, 1163531501);
            a = gg(a, b, c, d, k[13], 5, -1444681467);
            d = gg(d, a, b, c, k[2], 9, -51403784);
            c = gg(c, d, a, b, k[7], 14, 1735328473);
            b = gg(b, c, d, a, k[12], 20, -1926607734);
            a = hh(a, b, c, d, k[5], 4, -378558);
            d = hh(d, a, b, c, k[8], 11, -2022574463);
            c = hh(c, d, a, b, k[11], 16, 1839030562);
            b = hh(b, c, d, a, k[14], 23, -35309556);
            a = hh(a, b, c, d, k[1], 4, -1530992060);
            d = hh(d, a, b, c, k[4], 11, 1272893353);
            c = hh(c, d, a, b, k[7], 16, -155497632);
            b = hh(b, c, d, a, k[10], 23, -1094730640);
            a = hh(a, b, c, d, k[13], 4, 681279174);
            d = hh(d, a, b, c, k[0], 11, -358537222);
            c = hh(c, d, a, b, k[3], 16, -722521979);
            b = hh(b, c, d, a, k[6], 23, 76029189);
            a = hh(a, b, c, d, k[9], 4, -640364487);
            d = hh(d, a, b, c, k[12], 11, -421815835);
            c = hh(c, d, a, b, k[15], 16, 530742520);
            b = hh(b, c, d, a, k[2], 23, -995338651);
            a = ii(a, b, c, d, k[0], 6, -198630844);
            d = ii(d, a, b, c, k[7], 10, 1126891415);
            c = ii(c, d, a, b, k[14], 15, -1416354905);
            b = ii(b, c, d, a, k[5], 21, -57434055);
            a = ii(a, b, c, d, k[12], 6, 1700485571);
            d = ii(d, a, b, c, k[3], 10, -1894986606);
            c = ii(c, d, a, b, k[10], 15, -1051523);
            b = ii(b, c, d, a, k[1], 21, -2054922799);
            a = ii(a, b, c, d, k[8], 6, 1873313359);
            d = ii(d, a, b, c, k[15], 10, -30611744);
            c = ii(c, d, a, b, k[6], 15, -1560198380);
            b = ii(b, c, d, a, k[13], 21, 1309151649);
            a = ii(a, b, c, d, k[4], 6, -145523070);
            d = ii(d, a, b, c, k[11], 10, -1120210379);
            c = ii(c, d, a, b, k[2], 15, 718787259);
            b = ii(b, c, d, a, k[9], 21, -343485551);
            x[0] = (a + x[0]) | 0;
            x[1] = (b + x[1]) | 0;
            x[2] = (c + x[2]) | 0;
            x[3] = (d + x[3]) | 0;
        }
        function md5blk(s) {
            const md5blks = [];
            for (let i = 0; i < 64; i += 4) {
                md5blks[i >> 2] = s.charCodeAt(i) + (s.charCodeAt(i + 1) << 8) + (s.charCodeAt(i + 2) << 16) + (s.charCodeAt(i + 3) << 24);
            }
            return md5blks;
        }
        function md5blk_array(a) {
            const md5blks = [];
            for (let i = 0; i < 64; i += 4) {
                md5blks[i >> 2] = a[i] + (a[i + 1] << 8) + (a[i + 2] << 16) + (a[i + 3] << 24);
            }
            return md5blks;
        }
        function md51(s) {
            let n = s.length,
                state = [1732584193, -271733879, -1732584194, 271733878],
                i;
            let length, tail, tmp, lo, hi;
            for (i = 64; i <= n; i += 64) {
                md5cycle(state, md5blk(s.substring(i - 64, i)));
            }
            s = s.substring(i - 64);
            tail = new Array(64).fill(0);
            for (i = 0; i < s.length; i++) tail[i] = s.charCodeAt(i);
            tail[i] = 0x80;
            if (i > 55) {
                md5cycle(state, md5blk_array(tail));
                tail.fill(0);
            }
            // append length in bits
            length = n * 8;
            tail[56] = length & 0xff;
            tail[57] = (length >>> 8) & 0xff;
            tail[58] = (length >>> 16) & 0xff;
            tail[59] = (length >>> 24) & 0xff;
            md5cycle(state, md5blk_array(tail));
            return state;
        }
        function rhex(n) {
            let s = '', j = 0;
            for (; j < 4; j++) s += ((n >> (j * 8 + 4)) & 0x0f).toString(16) + ((n >> (j * 8)) & 0x0f).toString(16);
            return s;
        }
        function hex(x) {
            for (let i = 0; i < x.length; i++) x[i] = rhex(x[i]);
            return x.join('');
        }
        function md5(s) {
            return hex(md51(s));
        }

        // Handle file upload
        const wordlistFile = document.getElementById('wordlist-file');
        wordlistFile.addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(evt) {
                document.getElementById('wordlist').value = evt.target.result;
            };
            reader.readAsText(file);
        });

        document.getElementById('cracker-form').addEventListener('submit', async function(e) {
            e.preventDefault();
            const hash = document.getElementById('hash').value.trim().toLowerCase();
            const hashType = document.getElementById('hash-type').value;
            let wordlist = document.getElementById('wordlist').value.split(/\r?\n/).map(w => w.trim()).filter(Boolean);
            if (wordlist.length === 0) {
                document.getElementById('result').textContent = 'Please provide a wordlist.';
                document.getElementById('result').className = 'result fail';
                return;
            }
            document.getElementById('result').textContent = '';
            document.getElementById('result').className = 'result';
            let found = false;
            for (let i = 0; i < wordlist.length; i++) {
                document.getElementById('progress').textContent = `Trying: ${wordlist[i]} (${i+1}/${wordlist.length})`;
                let candidateHash = await hashString(wordlist[i], hashType);
                if (candidateHash === hash) {
                    found = true;
                    document.getElementById('result').textContent = `Password found: ${wordlist[i]}`;
                    document.getElementById('result').className = 'result success';
                    document.getElementById('progress').textContent = '';
                    break;
                }
            }
            if (!found) {
                document.getElementById('result').textContent = 'Password not found in wordlist.';
                document.getElementById('result').className = 'result fail';
                document.getElementById('progress').textContent = '';
            }
        });
    </script>
</body>
</html> 
