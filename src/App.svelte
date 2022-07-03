<script>

  import {userInfo} from './store'
  import Keycloak from "keycloak-js";

  let kc = new Keycloak({
    url: 'http://localhost:8080/', realm: 'master', clientId: 'app'
  });

  let logged_in = null;

  kc.init({onLoad: "check-sso"}).then((auth) => {
    if (auth) {
      logged_in = true;

      kc.loadUserInfo().then((user) => {
        user.token = kc.idToken
        userInfo.set(user)
      });
    }
  });

</script>

<h1>Svelte keycloak</h1>

{#if logged_in && $userInfo.preferred_username}

    <pre>{JSON.stringify($userInfo, null, 2)}</pre>

    You are logged in as {$userInfo.preferred_username}

    {kc.createLogoutUrl() + '?id_token_hint=' + $userInfo.token + '&post_logout_redirect_uri=' + encodeURIComponent(window.location.href)}
    <button
            on:click={() => {
			window.location.href=kc.createLogoutUrl() + '&id_token_hint=' + encodeURIComponent($userInfo.token) + '&post_logout_redirect_uri=' + encodeURIComponent(window.location.href);
		}}>Logout
    </button>

    <button
            on:click={() => {
			window.location.href=kc.createAccountUrl();
		}}> account
    </button>

{/if}

{#if !logged_in }
    You are not logged in
    <button
            on:click={() => {
			window.location.href=kc.createLoginUrl();
		}}>Login
    </button>

    try to register
    <button
            on:click={() => {
			window.location.href=kc.createRegisterUrl();
		}}>Register
    </button>

{/if}
