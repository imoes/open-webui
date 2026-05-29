<script lang="ts">
	import { getContext, onMount } from 'svelte';
	import { toast } from 'svelte-sonner';
	import { getOAuthConfig, updateOAuthConfig } from '$lib/apis/auths';
	import Switch from '$lib/components/common/Switch.svelte';
	import SensitiveInput from '$lib/components/common/SensitiveInput.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';

	const i18n = getContext('i18n');

	export let saveHandler: Function;

	let oauthConfig = null;

	const loadConfig = async () => {
		oauthConfig = await getOAuthConfig(localStorage.token);
	};

	const saveConfig = async () => {
		const res = await updateOAuthConfig(localStorage.token, oauthConfig).catch((err) => {
			toast.error(err);
			return null;
		});
		if (res) {
			oauthConfig = res;
			saveHandler();
		}
	};

	onMount(loadConfig);
</script>

{#if oauthConfig}
	<div class="flex flex-col gap-3 text-sm">
		<!-- Provider -->
		<div>
			<div class="mb-2 text-base font-medium">{$i18n.t('OIDC / SSO Provider')}</div>

			<div class="flex flex-col gap-2.5">
				<div class="flex w-full gap-2">
					<div class="flex-1">
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							{$i18n.t('Provider Name')}
						</div>
						<input
							class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
							type="text"
							placeholder="Keycloak"
							bind:value={oauthConfig.OAUTH_PROVIDER_NAME}
						/>
					</div>
				</div>

				<div class="flex w-full gap-2">
					<div class="flex-1">
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							{$i18n.t('Client ID')}
						</div>
						<input
							class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
							type="text"
							bind:value={oauthConfig.OAUTH_CLIENT_ID}
						/>
					</div>
					<div class="flex-1">
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							{$i18n.t('Client Secret')}
						</div>
						<SensitiveInput bind:value={oauthConfig.OAUTH_CLIENT_SECRET} placeholder="••••••••" />
					</div>
				</div>

				<div>
					<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
						{$i18n.t('Discovery URL (OIDC Provider URL)')}
					</div>
					<input
						class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
						type="text"
						placeholder="https://keycloak.example.com/realms/<realm>/.well-known/openid-configuration"
						bind:value={oauthConfig.OPENID_PROVIDER_URL}
					/>
				</div>

				<div class="flex w-full gap-2">
					<div class="flex-1">
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							{$i18n.t('Redirect URI')}
						</div>
						<input
							class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
							type="text"
							placeholder="https://open-webui.example.com/oauth/oidc/callback"
							bind:value={oauthConfig.OPENID_REDIRECT_URI}
						/>
					</div>
					<div class="flex-1">
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							{$i18n.t('End Session Endpoint (optional)')}
						</div>
						<input
							class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
							type="text"
							placeholder="https://keycloak.example.com/realms/<realm>/protocol/openid-connect/logout"
							bind:value={oauthConfig.OPENID_END_SESSION_ENDPOINT}
						/>
					</div>
				</div>

				<div class="flex w-full gap-2">
					<div class="flex-1">
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							{$i18n.t('Scopes')}
						</div>
						<input
							class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
							type="text"
							placeholder="openid email profile"
							bind:value={oauthConfig.OAUTH_SCOPES}
						/>
					</div>
					<div class="flex-1">
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							{$i18n.t('Token Endpoint Auth Method')}
						</div>
						<select
							class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
							bind:value={oauthConfig.OAUTH_TOKEN_ENDPOINT_AUTH_METHOD}
						>
							<option value={null}>{$i18n.t('(default)')}</option>
							<option value="client_secret_post">client_secret_post</option>
							<option value="client_secret_basic">client_secret_basic</option>
							<option value="none">none (PKCE)</option>
						</select>
					</div>
				</div>
			</div>
		</div>

		<hr class="border-gray-100 dark:border-gray-850" />

		<!-- Access Control -->
		<div>
			<div class="mb-2 text-base font-medium">{$i18n.t('Access Control')}</div>

			<div class="flex flex-col gap-2.5">
				<div class="flex w-full items-center justify-between">
					<div class="text-sm">{$i18n.t('Allow OAuth Sign-up')}</div>
					<Switch bind:state={oauthConfig.ENABLE_OAUTH_SIGNUP} />
				</div>

				<div class="flex w-full items-center justify-between">
					<Tooltip
						content={$i18n.t(
							'Merge existing local accounts with the same email on first OAuth login'
						)}
					>
						<div class="text-sm">{$i18n.t('Merge Accounts by Email')}</div>
					</Tooltip>
					<Switch bind:state={oauthConfig.OAUTH_MERGE_ACCOUNTS_BY_EMAIL} />
				</div>

				<div>
					<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
						<Tooltip
							content={$i18n.t(
								'Comma-separated list of allowed email domains. Use * to allow all domains.'
							)}
						>
							{$i18n.t('Allowed Domains')}
						</Tooltip>
					</div>
					<input
						class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
						type="text"
						placeholder="*"
						bind:value={oauthConfig.OAUTH_ALLOWED_DOMAINS}
					/>
				</div>
			</div>
		</div>

		<hr class="border-gray-100 dark:border-gray-850" />

		<!-- Claims Mapping -->
		<div>
			<div class="mb-2 text-base font-medium">{$i18n.t('Claims Mapping')}</div>

			<div class="grid grid-cols-2 gap-2">
				<div>
					<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
						{$i18n.t('Username Claim')}
					</div>
					<input
						class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
						type="text"
						placeholder="preferred_username"
						bind:value={oauthConfig.OAUTH_USERNAME_CLAIM}
					/>
				</div>
				<div>
					<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
						{$i18n.t('Email Claim')}
					</div>
					<input
						class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
						type="text"
						placeholder="email"
						bind:value={oauthConfig.OAUTH_EMAIL_CLAIM}
					/>
				</div>
				<div>
					<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
						{$i18n.t('Picture Claim')}
					</div>
					<input
						class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
						type="text"
						placeholder="picture"
						bind:value={oauthConfig.OAUTH_PICTURE_CLAIM}
					/>
				</div>
				<div>
					<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
						{$i18n.t('Subject Claim (optional)')}
					</div>
					<input
						class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
						type="text"
						placeholder="sub"
						bind:value={oauthConfig.OAUTH_SUB_CLAIM}
					/>
				</div>
			</div>

			<div class="mt-2.5 flex flex-col gap-1.5">
				<div class="flex w-full items-center justify-between">
					<div class="text-sm">{$i18n.t('Update Picture on Login')}</div>
					<Switch bind:state={oauthConfig.OAUTH_UPDATE_PICTURE_ON_LOGIN} />
				</div>
				<div class="flex w-full items-center justify-between">
					<div class="text-sm">{$i18n.t('Update Name on Login')}</div>
					<Switch bind:state={oauthConfig.OAUTH_UPDATE_NAME_ON_LOGIN} />
				</div>
				<div class="flex w-full items-center justify-between">
					<div class="text-sm">{$i18n.t('Update Email on Login')}</div>
					<Switch bind:state={oauthConfig.OAUTH_UPDATE_EMAIL_ON_LOGIN} />
				</div>
			</div>
		</div>

		<hr class="border-gray-100 dark:border-gray-850" />

		<!-- Role Management -->
		<div>
			<div class="mb-2 flex items-center justify-between text-base font-medium">
				<span>{$i18n.t('Role Management')}</span>
				<Switch bind:state={oauthConfig.ENABLE_OAUTH_ROLE_MANAGEMENT} />
			</div>

			{#if oauthConfig.ENABLE_OAUTH_ROLE_MANAGEMENT}
				<div class="flex flex-col gap-2.5">
					<div>
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							<Tooltip
								content={$i18n.t(
									'Claim in the token that contains the user roles, e.g. realm_access.roles'
								)}
							>
								{$i18n.t('Roles Claim')}
							</Tooltip>
						</div>
						<input
							class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
							type="text"
							placeholder="realm_access.roles"
							bind:value={oauthConfig.OAUTH_ROLES_CLAIM}
						/>
					</div>
					<div class="flex gap-2">
						<div class="flex-1">
							<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
								<Tooltip content={$i18n.t('Comma-separated list of roles that grant user access')}>
									{$i18n.t('Allowed Roles')}
								</Tooltip>
							</div>
							<input
								class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
								type="text"
								placeholder="user,admin"
								bind:value={oauthConfig.OAUTH_ALLOWED_ROLES}
							/>
						</div>
						<div class="flex-1">
							<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
								<Tooltip content={$i18n.t('Comma-separated list of roles that grant admin access')}>
									{$i18n.t('Admin Roles')}
								</Tooltip>
							</div>
							<input
								class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
								type="text"
								placeholder="admin"
								bind:value={oauthConfig.OAUTH_ADMIN_ROLES}
							/>
						</div>
					</div>
				</div>
			{/if}
		</div>

		<hr class="border-gray-100 dark:border-gray-850" />

		<!-- Group Management -->
		<div>
			<div class="mb-2 flex items-center justify-between text-base font-medium">
				<span>{$i18n.t('Group Synchronization')}</span>
				<Switch bind:state={oauthConfig.ENABLE_OAUTH_GROUP_MANAGEMENT} />
			</div>

			{#if oauthConfig.ENABLE_OAUTH_GROUP_MANAGEMENT}
				<div class="flex flex-col gap-2.5">
					<div>
						<div class="mb-1 text-xs font-medium text-gray-500 dark:text-gray-400">
							{$i18n.t('Groups Claim')}
						</div>
						<input
							class="w-full rounded-lg border border-gray-100 bg-gray-50 px-3 py-1.5 text-sm dark:border-gray-850 dark:bg-gray-850"
							type="text"
							placeholder="groups"
							bind:value={oauthConfig.OAUTH_GROUPS_CLAIM}
						/>
					</div>
					<div class="flex w-full items-center justify-between">
						<div class="text-sm">{$i18n.t('Auto-Create Missing Groups')}</div>
						<Switch bind:state={oauthConfig.ENABLE_OAUTH_GROUP_CREATION} />
					</div>
				</div>
			{/if}
		</div>

		<div class="flex justify-end pt-2 pb-4">
			<button
				class="rounded-lg bg-black px-4 py-2 text-sm font-medium text-white hover:bg-gray-800 dark:bg-white dark:text-black dark:hover:bg-gray-100"
				on:click={saveConfig}
			>
				{$i18n.t('Save')}
			</button>
		</div>
	</div>
{:else}
	<div class="flex h-full items-center justify-center">
		<div class="text-sm text-gray-400">{$i18n.t('Loading...')}</div>
	</div>
{/if}
