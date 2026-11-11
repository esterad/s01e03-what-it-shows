<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem the reports show. A report is placed by what it shows first (the desk's
  // one-day moves are no swap), then by the path its date took (causes.ts).
  type Problem = 'parser' | 'hint' | 'shift' | 'unplaced';
  type Ticket = { label: string; href: string; title: string };

  let { ticket, tickets }: { ticket: number; tickets: Partial<Record<Problem, Ticket>> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = (reports as Report[]).filter((r) => r.mergedInto === ticket).length;

  const problemOf = (r: Report): Problem =>
    notes.oneDayMove.includes(r.id) ? 'shift' : (causeOf(r, tenantOf(r)) ?? 'unplaced');

  const WHAT: Record<Problem, string> = {
    parser: 'day and month swapped by the shared parser',
    hint: 'day and month swapped by the booking-v3 form hint',
    shift: 'date moves by exactly one day',
    unplaced: 'no tenant on file',
  };

  const groups = $derived.by(() => {
    const by = new Map<Problem, Report[]>();
    for (const r of merged) by.set(problemOf(r), [...(by.get(problemOf(r)) ?? []), r]);
    return [...by.entries()]
      .map(([problem, rs]) => ({
        problem,
        count: rs.length,
        tenants: new Set(rs.map((r) => r.tenant)).size,
        channels: [...new Set(rs.map((r) => r.channel))].sort().join(', '),
      }))
      .sort((a, b) => b.count - a.count);
  });
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ({rows} export rows, counted once by id) ·
  <strong>{groups.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>what the reports show</th><th>reports</th><th>tenants</th><th>channels</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.problem)}
      <tr>
        <td><code>{g.problem}</code></td>
        <td>{WHAT[g.problem]}</td>
        <td><strong>{g.count}</strong></td>
        <td>{g.tenants}</td>
        <td class="muted">{g.channels}</td>
        <td>
          {#if tickets[g.problem]}
            <a href={tickets[g.problem]?.href} title={tickets[g.problem]?.title}>{tickets[g.problem]?.label}</a>
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
