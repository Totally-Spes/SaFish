<script>
  import { onMount } from "svelte";
import { element } from "svelte/internal";

  // text fields variables
  let username
  let password
  let api_link
  let logged_in = false


  // Show mobile icon and display menu
  let showMobileMenu = false;

  // Mobile menu click event handler
  const handleMobileIconClick = () => (showMobileMenu = !showMobileMenu);

  // Media match query handler
  const mediaQueryHandler = e => {
    // Reset mobile state
    if (!e.matches) {
      showMobileMenu = false;
    }
  };

  // Attach media query listener on mount hook
  onMount(() => {
    const mediaListener = window.matchMedia("(max-width: 767px)");

    mediaListener.addListener(mediaQueryHandler);
  });

  /**
* Secure Hash Algorithm (SHA256)
* http://www.webtoolkit.info/
* Original code by Angel Marin, Paul Johnston
**/

  function SHA256(s){
    var chrsz = 8;
    var hexcase = 0;

    function safe_add (x, y) {
    var lsw = (x & 0xFFFF) + (y & 0xFFFF);
    var msw = (x >> 16) + (y >> 16) + (lsw >> 16);
    return (msw << 16) | (lsw & 0xFFFF);
  }

  function S (X, n) { return ( X >>> n ) | (X << (32 - n)); }
  function R (X, n) { return ( X >>> n ); }
  function Ch(x, y, z) { return ((x & y) ^ ((~x) & z)); }
  function Maj(x, y, z) { return ((x & y) ^ (x & z) ^ (y & z)); }
  function Sigma0256(x) { return (S(x, 2) ^ S(x, 13) ^ S(x, 22)); }
  function Sigma1256(x) { return (S(x, 6) ^ S(x, 11) ^ S(x, 25)); }
  function Gamma0256(x) { return (S(x, 7) ^ S(x, 18) ^ R(x, 3)); }
  function Gamma1256(x) { return (S(x, 17) ^ S(x, 19) ^ R(x, 10)); }

  function core_sha256 (m, l) {
    var K = new Array(0x428A2F98, 0x71374491, 0xB5C0FBCF, 0xE9B5DBA5, 0x3956C25B, 0x59F111F1, 0x923F82A4, 0xAB1C5ED5, 0xD807AA98, 0x12835B01, 0x243185BE, 0x550C7DC3, 0x72BE5D74, 0x80DEB1FE, 0x9BDC06A7, 0xC19BF174, 0xE49B69C1, 0xEFBE4786, 0xFC19DC6, 0x240CA1CC, 0x2DE92C6F, 0x4A7484AA, 0x5CB0A9DC, 0x76F988DA, 0x983E5152, 0xA831C66D, 0xB00327C8, 0xBF597FC7, 0xC6E00BF3, 0xD5A79147, 0x6CA6351, 0x14292967, 0x27B70A85, 0x2E1B2138, 0x4D2C6DFC, 0x53380D13, 0x650A7354, 0x766A0ABB, 0x81C2C92E, 0x92722C85, 0xA2BFE8A1, 0xA81A664B, 0xC24B8B70, 0xC76C51A3, 0xD192E819, 0xD6990624, 0xF40E3585, 0x106AA070, 0x19A4C116, 0x1E376C08, 0x2748774C, 0x34B0BCB5, 0x391C0CB3, 0x4ED8AA4A, 0x5B9CCA4F, 0x682E6FF3, 0x748F82EE, 0x78A5636F, 0x84C87814, 0x8CC70208, 0x90BEFFFA, 0xA4506CEB, 0xBEF9A3F7, 0xC67178F2);
    var HASH = new Array(0x6A09E667, 0xBB67AE85, 0x3C6EF372, 0xA54FF53A, 0x510E527F, 0x9B05688C, 0x1F83D9AB, 0x5BE0CD19);
    var W = new Array(64);
    var a, b, c, d, e, f, g, h, i, j;
    var T1, T2;

    m[l >> 5] |= 0x80 << (24 - l % 32);
    m[((l + 64 >> 9) << 4) + 15] = l;

    for ( var i = 0; i<m.length; i+=16 ) {
      a = HASH[0];
      b = HASH[1];
      c = HASH[2];
      d = HASH[3];
      e = HASH[4];
      f = HASH[5];
      g = HASH[6];
      h = HASH[7];

      for ( var j = 0; j<64; j++) {
        if (j < 16) W[j] = m[j + i];
        else W[j] = safe_add(safe_add(safe_add(Gamma1256(W[j - 2]), W[j - 7]), Gamma0256(W[j - 15])), W[j - 16]);

        T1 = safe_add(safe_add(safe_add(safe_add(h, Sigma1256(e)), Ch(e, f, g)), K[j]), W[j]);
        T2 = safe_add(Sigma0256(a), Maj(a, b, c));

        h = g;
        g = f;
        f = e;
        e = safe_add(d, T1);
        d = c;
        c = b;
        b = a;
        a = safe_add(T1, T2);
      }

      HASH[0] = safe_add(a, HASH[0]);
      HASH[1] = safe_add(b, HASH[1]);
      HASH[2] = safe_add(c, HASH[2]);
      HASH[3] = safe_add(d, HASH[3]);
      HASH[4] = safe_add(e, HASH[4]);
      HASH[5] = safe_add(f, HASH[5]);
      HASH[6] = safe_add(g, HASH[6]);
      HASH[7] = safe_add(h, HASH[7]);
      }
      return HASH;
    }

    function str2binb (str) {
    var bin = Array();
    var mask = (1 << chrsz) - 1;
    for(var i = 0; i < str.length * chrsz; i += chrsz) {
    bin[i>>5] |= (str.charCodeAt(i / chrsz) & mask) << (24 - i % 32);
  }
  return bin;
  }

  function Utf8Encode(string) {
  string = string.replace(/\r\n/g,'\n');
  var utftext = '';

  for (var n = 0; n < string.length; n++) {

  var c = string.charCodeAt(n);

  if (c < 128) {
  utftext += String.fromCharCode(c);
  }
  else if((c > 127) && (c < 2048)) {
  utftext += String.fromCharCode((c >> 6) | 192);
  utftext += String.fromCharCode((c & 63) | 128);
  }
  else {
  utftext += String.fromCharCode((c >> 12) | 224);
  utftext += String.fromCharCode(((c >> 6) & 63) | 128);
  utftext += String.fromCharCode((c & 63) | 128);
  }

  }

  return utftext;
  }

  function binb2hex (binarray) {
  var hex_tab = hexcase ? '0123456789ABCDEF' : '0123456789abcdef';
  var str = '';
  for(var i = 0; i < binarray.length * 4; i++) {
  str += hex_tab.charAt((binarray[i>>2] >> ((3 - i % 4)*8+4)) & 0xF) +
  hex_tab.charAt((binarray[i>>2] >> ((3 - i % 4)*8 )) & 0xF);
  }
  return str;
  }

  s = Utf8Encode(s);
  return binb2hex(core_sha256(str2binb(s), s.length * chrsz));
  }


  // Login function to be called from the login button with username and password
  async function login() {
      console.log("Login");
      username = document.getElementsByName("user")[0].value;
      console.log(username);
      password = document.getElementsByName("pass")[0].value;
      console.log(password);
      var hash = SHA256(password);
      console.log(hash);
      let response = await fetch(`http://localhost:5001/api/account/login/${username}/${hash}`);
      if (response.status == 200) {
          logged_in = true;
      } 
      else {
          alert("Invalid username or password");
      }
  }

  function logout() {
      logged_in = false;
  }

</script>
  
  <nav>
    <div class="inner">
      <div on:click={handleMobileIconClick} class={`mobile-icon${showMobileMenu ? ' active' : ''}`}>
        <div class="middle-line"></div>
      </div>
      <ul class={`navbar-list${showMobileMenu ? ' mobile' : ''}`}>
          <li>
            <h2 style="color:white">CFish</h2>
          </li>

        {#if (!logged_in)}
          
          <li>
            <input type="username" id="user" name="user" required
              minlength="4" maxlength="8" size="10" placeholder="username">
          </li>
          <li>
            <input type="password" id="pass" name="pass" required
              minlength="8" maxlength="12" size="12" placeholder="password">
          </li>
          <li>
            <h2 style="color:white" on:click={login}>Login</h2>
          </li>
        {:else}
          <li>
            <img src="fishrod.png" style="height:auto width:auto" alt="fish">
          </li>
          <li>
            <img src="fishy.png" style="height:5% width:5%" alt="move">
          </li>
          <li>
            <h2 style="color:white" on:click={logout}>Logout</h2>
          </li>
        }
        {/if}


          
      </ul>
    </div>
  </nav>
  
  <style>
    *{
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      font-family: "Ubuntu",sans-serif;
    }

    body{
      background: "background_manche.png" no-repeat center;
      background-size: cover;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
    }


    nav {
      background-color: rgba(0, 0, 0, 0.8);
      font-family: "Helvetica Neue", "Helvetica", "Arial", sans-serif;
      height: 45px;
    }
  
    .inner {
      max-width: 980px;
      padding-left: 20px;
      padding-right: 20px;
      margin: auto;
      box-sizing: border-box;
      display: flex;
      align-items: center;
      height: 100%;
    }
  
    .mobile-icon {
      width: 25px;
      height: 14px;
      position: relative;
      cursor: pointer;
    }
  
    .mobile-icon:after,
    .mobile-icon:before,
    .middle-line {
      content: "";
      position: absolute;
      width: 100%;
      height: 2px;
      background-color: #fff;
      transition: all 0.4s;
      transform-origin: center;
    }
  
    .mobile-icon:before,
    .middle-line {
      top: 0;
    }
  
    .mobile-icon:after,
    .middle-line {
      bottom: 0;
    }
  
    .mobile-icon:before {
      width: 66%;
    }
  
    .mobile-icon:after {
      width: 33%;
    }
  
    .middle-line {
      margin: auto;
    }
  
    .mobile-icon:hover:before,
    .mobile-icon:hover:after,
    .mobile-icon.active:before,
    .mobile-icon.active:after,
    .mobile-icon.active .middle-line {
      width: 100%;
    }
  
    .mobile-icon.active:before,
    .mobile-icon.active:after {
      top: 50%;
      transform: rotate(-45deg);
    }
  
    .mobile-icon.active .middle-line {
      transform: rotate(45deg);
    }
  
    .navbar-list {
      display: none;
      width: 100%;
      justify-content: space-between;
      margin: 0;
      padding: 0 40px;
    }
  
    .navbar-list.mobile {
      background-color: rgba(0, 0, 0, 0.8);
      position: fixed;
      display: block;
      height: calc(100% - 45px);
      bottom: 0;
      left: 0;
    }
  
    .navbar-list li {
      list-style-type: none;
      position: relative;
    }
  
    .navbar-list li:before {
      content: "";
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 1px;
      background-color: #424245;
    }
  
    .navbar-list a {
      color: #fff;
      text-decoration: none;
      display: flex;
      height: 45px;
      align-items: center;
      padding: 0 10px;
      font-size: 13px;
    }
  
    @media only screen and (min-width: 767px) {
      .mobile-icon {
        display: none;
      }
  
      .navbar-list {
        display: flex;
        padding: 0;
      }
  
      .navbar-list a {
        display: inline-flex;
      }
    }
  </style>
  