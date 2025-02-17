<script lang="ts">
    import { page } from '$app/stores';
    import {
        CardGrid,
        DropList,
        DropListItem,
        DropListLink,
        EmptySearch,
        Heading,
        PaginationInline
    } from '$lib/components';
    import { Pill } from '$lib/elements';
    import { Button } from '$lib/elements/forms';
    import {
        TableBody,
        TableCell,
        TableCellHead,
        TableCellText,
        TableHeader,
        TableRow,
        TableScroll
    } from '$lib/elements/table';
    import { toLocaleDate } from '$lib/helpers/date';
    import type { Invoice, InvoiceList } from '$lib/sdk/billing';
    import { sdk } from '$lib/stores/sdk';
    import { VARS } from '$lib/system';
    import { Query } from '@appwrite.io/console';
    import { onMount } from 'svelte';
    import RetryPaymentModal from './retryPaymentModal.svelte';
    import { trackEvent } from '$lib/actions/analytics';

    let showDropdown = [];
    let showRetryModal = false;
    let selectedInvoice: Invoice | null = null;

    let offset = 0;
    let invoiceList: InvoiceList = {
        invoices: [],
        total: 0
    };

    const limit = 5;
    const endpoint = VARS.APPWRITE_ENDPOINT ?? `${$page.url.origin}/v1`;

    onMount(request);

    async function request() {
        invoiceList = await sdk.forConsole.billing.listInvoices($page.params.organization, [
            Query.limit(limit),
            Query.offset(offset),
            Query.orderDesc('$createdAt')
        ]);
    }

    $: if (offset !== null) {
        request();
    }
</script>

<CardGrid>
    <Heading tag="h2" size="6">Payment history</Heading>

    <p class="text">
        Transaction history for this organization. Download invoices for more details about your
        payments.
    </p>
    <svelte:fragment slot="aside">
        {#if invoiceList.total - 1 > 0}
            <TableScroll noMargin transparent>
                <TableHeader>
                    <TableCellHead width={100}>Due Date</TableCellHead>
                    <TableCellHead width={80}>Status</TableCellHead>
                    <TableCellHead width={100}>Amount Due</TableCellHead>
                    <TableCellHead width={40} />
                </TableHeader>
                <TableBody>
                    {#each invoiceList?.invoices as invoice, i}
                        {@const status = invoice.status}
                        {#if i !== 0}
                            <TableRow>
                                <TableCellText title="date">
                                    {toLocaleDate(invoice.dueAt)}
                                </TableCellText>

                                <TableCell title="status">
                                    <Pill
                                        danger={status === 'overdue' ||
                                            status === 'failed' ||
                                            status === 'requires_authentication'}
                                        success={status === 'paid' || status === 'succeeded'}
                                        warning={status === 'pending'}>
                                        {status === 'requires_authentication' ? 'failed' : status}
                                    </Pill>
                                </TableCell>
                                <TableCellText title="due">${invoice.amount}</TableCellText>
                                <TableCell showOverflow>
                                    <DropList
                                        bind:show={showDropdown[i]}
                                        placement="bottom-start"
                                        noArrow>
                                        <Button
                                            round
                                            text
                                            noMargin
                                            ariaLabel="More options"
                                            on:click={() => {
                                                showDropdown[i] = !showDropdown[i];
                                            }}>
                                            <span class="icon-dots-horizontal" aria-hidden="true" />
                                        </Button>
                                        <svelte:fragment slot="list">
                                            <DropListLink
                                                icon="external-link"
                                                external
                                                href={`${endpoint}/organizations/${$page.params.organization}/invoices/${invoice.$id}/view`}
                                                on:click={() =>
                                                    (showDropdown[i] = !showDropdown[i])}
                                                event="view_invoice">
                                                View invoice
                                            </DropListLink>
                                            <DropListLink
                                                icon="download"
                                                href={`${endpoint}/organizations/${$page.params.organization}/invoices/${invoice.$id}/download`}
                                                on:click={() => {
                                                    showDropdown[i] = !showDropdown[i];
                                                }}
                                                event="download_invoice">
                                                Download PDF
                                            </DropListLink>
                                            {#if status === 'overdue' || status === 'failed'}
                                                <DropListItem
                                                    icon="refresh"
                                                    on:click={() => {
                                                        selectedInvoice = invoice;
                                                        showRetryModal = true;
                                                        showDropdown[i] = !showDropdown[i];
                                                        trackEvent(`click_retry_payment`, {
                                                            from: 'button',
                                                            source: 'billing_invoice_menu'
                                                        });
                                                    }}>
                                                    Retry payment
                                                </DropListItem>
                                            {/if}
                                        </svelte:fragment>
                                    </DropList>
                                </TableCell>
                            </TableRow>
                        {/if}
                    {/each}
                </TableBody>
            </TableScroll>
            <div class="u-flex u-main-space-between">
                <p class="text">Total results: {invoiceList?.total - 1 ?? 0}</p>
                <PaginationInline {limit} bind:offset sum={invoiceList?.total - 1 ?? 0} hidePages />
            </div>
        {:else}
            <EmptySearch hidePagination>
                <p class="text u-text-center">
                    You have no payment history. After you receive your first invoice, you'll see it
                    here.
                </p>
            </EmptySearch>
        {/if}
    </svelte:fragment>
</CardGrid>

{#if selectedInvoice}
    <RetryPaymentModal bind:show={showRetryModal} bind:invoice={selectedInvoice} />
{/if}
