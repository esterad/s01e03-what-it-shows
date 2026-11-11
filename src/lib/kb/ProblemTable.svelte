<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem in the ticket, counted with the desk's own view as exported: the cause
  // groups of causes.ts, and the reports the desk read as a one-day move (desk-notes.json), which
  // stay counted where they are filed and also get a row of their own.
  type Problem = 'parser' | 'hint' | 'unplaced' | 'shift';
  type Ticket = { label: string; href: string; title: string };

  let { ticket, tickets }: { ticket: number; tickets: Partial<Record<Problem, Ticket>> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = (reports as Report[]).filter((r) => r.mergedInto === ticket).length;
  const groups = $derived(groupByCause(merged, tenantOf));

  const WHAT: Record<Problem, string> = {
    parser: 'day and month swapped by the shared parser',
    hint: 'day and month swapped by the booking-v3 form hint',
    unplaced: 'no tenant on file',
    shift: 'date moves by exactly one day',
  };

  const problems = $derived.by(() => {
    const out: { problem: Problem; count: number; tenants: number; also?: string }[] = groups.map((g) => ({
      problem: g.cause,
      count: g.reports.length,
      tenants: g.tenants,
    }));
    for (const g of groups) {
      const moved = g.reports.filter((r) => notes.oneDayMove.includes(r.id));
      if (moved.length)
        out.push({
          problem: 'shift',
          count: readAs(g, notes.oneDayMove),
          tenants: new Set(moved.map((r) => r.tenant)).size,
          also: g.cause,
        });
    }
    return out;
  });
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ({rows} export rows, counted once by id) ·
  <strong>{problems.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>what the reports show</th><th>reports</th><th>tenants</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each problems as p (p.problem)}
      <tr>
        <td><code>{p.problem}</code></td>
        <td>{WHAT[p.problem]}{#if p.also}<br /><span class="muted">also counted under <code>{p.also}</code>, where the desk files them</span>{/if}</td>
        <td><strong>{p.count}</strong></td>
        <td>{p.tenants}</td>
        <td>
          {#if tickets[p.problem]}
            <a href={tickets[p.problem]?.href} title={tickets[p.problem]?.title}>{tickets[p.problem]?.label}</a>
          {:else}
            <span class="none">no ticket</span>
          {/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .none { color: var(--warn); font-weight: 600; }
</style>
