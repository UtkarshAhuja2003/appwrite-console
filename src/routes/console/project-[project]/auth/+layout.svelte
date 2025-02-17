<script lang="ts">
    import { goto } from '$app/navigation';
    import { page } from '$app/stores';
    import { registerCommands, updateCommandGroupRanks } from '$lib/commandCenter';
    import { readOnly } from '$lib/stores/billing';
    import { GRACE_PERIOD_OVERRIDE } from '$lib/system';
    import { project } from '../store';
    import { showCreateUser } from './+page.svelte';
    import { showCreateTeam } from './teams/+page.svelte';

    $: $registerCommands([
        {
            label: 'Create user',
            callback: async () => {
                if (!$page.url.pathname.endsWith('auth')) {
                    await goto(`/console/project-${$project.$id}/auth`);
                }
                showCreateUser.set(true);
            },
            keys: $page.url.pathname.endsWith('auth') ? ['c'] : ['c', 'u'],
            group: 'users',
            icon: 'plus',
            rank: $page.url.pathname.endsWith('auth') ? 10 : 0,
            disabled: $readOnly && !GRACE_PERIOD_OVERRIDE
        },
        {
            label: 'Create team',
            callback: async () => {
                if (!$page.url.pathname.endsWith('teams')) {
                    await goto(`/console/project-${$project.$id}/auth/teams`);
                }
                showCreateTeam.set(true);
            },
            keys: $page.url.pathname.endsWith('teams') ? ['c'] : ['c', 't'],

            group: 'teams',
            icon: 'plus',
            rank: $page.url.pathname.endsWith('teams') ? 10 : 0,
            disabled: $readOnly && !GRACE_PERIOD_OVERRIDE
        },
        {
            label: 'Go to teams',
            keys: ['g', 't'],
            callback() {
                goto(`/console/project-${$project.$id}/auth/teams`);
            },
            group: 'navigation',
            rank: 1,
            disabled: $page.url.pathname.endsWith('teams')
        },
        {
            label: 'Go to usage',
            keys: ['g', 'u'],
            callback() {
                goto(`/console/project-${$project.$id}/auth/usage`);
            },
            group: 'navigation',
            rank: 1,
            disabled: $page.url.pathname.endsWith('usage')
        },
        {
            label: 'Go to security',
            keys: ['g', 'e'],
            callback() {
                goto(`/console/project-${$project.$id}/auth/security`);
            },
            group: 'navigation',
            rank: 1,
            disabled: $page.url.pathname.endsWith('security')
        },
        {
            label: 'Go to settings',
            keys: ['g', 's'],
            callback() {
                goto(`/console/project-${$project.$id}/auth/settings`);
            },
            group: 'navigation',
            rank: 1,
            disabled: $page.url.pathname.endsWith('settings')
        }
    ]);

    // To prioritize the groups!
    $: $updateCommandGroupRanks({ users: 300, teams: 200, security: 100, navigation: 50 });
</script>

<svelte:head>
    <title>Auth - Appwrite</title>
</svelte:head>

<slot />
