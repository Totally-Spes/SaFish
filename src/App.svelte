<script>

	import Map from './Map.svelte';
	import MapMarker from './MapMarker.svelte';
	import Menu from './Menu.svelte'
	

    
</script>

<!-- LOGIN FORM -->
<!-- <form action="#" class="login-form" style="display:none;disabled:true">
	<h1>Sign In</h1>
	<div class="textb">
	<input type="text" required>
	<div class.login-form{
		width: calc(100% - 20px);
		max-width: 500px;
		background-color: #fff;
		padding: 50px 40px;
	}

	.login-form h1{
		text-align: center;
		margin: 48px 0;
		font-size: 36px;
		color: #111;
		font-size: 36px;
	}

	.textb{
		margin-bottom: 12px;
		position: relative;
	}

	.textb input{
	width: 100%;
	height: 70px;
	border: none;
	background-color: #ededed;
	border-radius: 4px;
	color: #333;
	font-size: 18px;
	font-weight: 700;
	padding: 14px 60px 0 10px;
	}

	.placeholder{
		text-transform: uppercase;
		font-size: 14px;
		font-weight: 700;
		position: absolute;
		line-height: 70px;
		top: 0;
		left: 20px;
		color: #9d9d9d;
		user-select: none;
		pointer-events: none;
	}

	.show-password{
		position: absolute;
		right: 20px;
		line-height: 70px;
		color: #9d9d9d;
		font-size: 22px;
		cursor: pointer;
	}

	.textb input:focus ~ .placeholder,
	.textb input:valid ~ .placeholder{
		left: 10px;
		top: -16px;
	}

	.checkbox{
		display: flex;
		align-items: center;
		position: relative;
		font-size: 18px;
		font-weight: 500;
		color: #333;
		margin: 20px 0;
	}

	.checkbox input{
		width: 24px;
		height: 24px;
		-webkit-appearance: none;
		background-color: #ededed;
		border-radius: 4px;
		margin-right: 10px;
		cursor: pointer;
	}

	.checkbox .fa-check{
		position: absolute;
		color: #fff;
		font-size: 14px;
		width: 24px;
		text-align: center;
		user-select: none;
		pointer-events: none;
		opacity: 0;
	}

	.checkbox input:checked{
		background-color: #bc252a;
	}

	.checkbox input:checked + .fa-check{
		opacity: 1;
	}

	.btn{
		display: block;
		width: 100px;
		height: 100px;
		margin: 40px auto;
		background-color: #bc252a;
		color: #fff;
		font-size: 40px;
		border: none;
		border-radius: 30px;
		cursor: pointer;
		transition: .3s linear;
	}

	.btn:hover{
		opacity: .85;
	}

	.login-form a{
		display: block;
		text-align: center;
		text-decoration: none;
		font-size: 16px;
		color: #555;
		text-transform: uppercase;
		font-weight: 500;
		margin-bottom: 8px;
		transition: .2s linear;
	}

	.login-form a:hover{
		color: #111;
	}

	.btn:disabled{
		background-color: transparent;
		color: #ddd;
		border: 2px solid;
	}="placeholder">Username</div>
	</div>

	<div class="textb">
	<input type="password" required>
	<div class="placeholder">Password</div>
	<div class="show-password fas fa-eye-slash"></div>
	</div>

	<div class="checkbox">
	<input type="checkbox">
	<div class="fas fa-check"></div>
	Stay signed in
	</div>

	<button class="btn fas fa-arrow-right" disabled></button>
	<a href="#">Can't Sign in?</a>
	<a href="#">Create Account</a>
</form> -->
<Map lat={40} lon={0} zoom={3.5} >
</Map>
<div class="map-container">
	<Map lat={40} lon={0} zoom={3.5} >
	</Map>
</div>

<style>
	/* beginning of classes for login form */

	/* .login-form{
		width: calc(100% - 20px);
		max-width: 500px;
		background-color: #fff;
		padding: 50px 40px;
	}

	.login-form h1{
		text-align: center;
		margin: 48px 0;
		font-size: 36px;
		color: #111;
		font-size: 36px;
	}

	.textb{
		margin-bottom: 12px;
		position: relative;
	}

	.textb input{
	width: 100%;
	height: 70px;
	border: none;
	background-color: #ededed;
	border-radius: 4px;
	color: #333;
	font-size: 18px;
	font-weight: 700;
	padding: 14px 60px 0 10px;
	}

	.placeholder{
		text-transform: uppercase;
		font-size: 14px;
		font-weight: 700;
		position: absolute;
		line-height: 70px;
		top: 0;
		left: 20px;
		color: #9d9d9d;
		user-select: none;
		pointer-events: none;
	}

	.show-password{
		position: absolute;
		right: 20px;
		line-height: 70px;
		color: #9d9d9d;
		font-size: 22px;
		cursor: pointer;
	}

	.textb input:focus ~ .placeholder,
	.textb input:valid ~ .placeholder{
		left: 10px;
		top: -16px;
	}

	.checkbox{
		display: flex;
		align-items: center;
		position: relative;
		font-size: 18px;
		font-weight: 500;
		color: #333;
		margin: 20px 0;
	}

	.checkbox input{
		width: 24px;
		height: 24px;
		-webkit-appearance: none;
		background-color: #ededed;
		border-radius: 4px;
		margin-right: 10px;
		cursor: pointer;
	}

	.checkbox .fa-check{
		position: absolute;
		color: #fff;
		font-size: 14px;
		width: 24px;
		text-align: center;
		user-select: none;
		pointer-events: none;
		opacity: 0;
	}

	.checkbox input:checked{
		background-color: #bc252a;
	}

	.checkbox input:checked + .fa-check{
		opacity: 1;
	}

	.btn{
		display: block;
		width: 100px;
		height: 100px;
		margin: 40px auto;
		background-color: #bc252a;
		color: #fff;
		font-size: 40px;
		border: none;
		border-radius: 30px;
		cursor: pointer;
		transition: .3s linear;
	}

	.btn:hover{
		opacity: .85;
	}

	.login-form a{
		display: block;
		text-align: center;
		text-decoration: none;
		font-size: 16px;
		color: #555;
		text-transform: uppercase;
		font-weight: 500;
		margin-bottom: 8px;
		transition: .2s linear;
	}

	.login-form a:hover{
		color: #111;
	}

	.btn:disabled{
		background-color: transparent;
		color: #ddd;
		border: 2px solid;
	} */

/* end of classes for login form */
</style>