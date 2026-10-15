---
name: admin-members-and-groups
description: Help a Lighthouse organization admin manage members, invitations, roles, business groups and organization settings, confirming every change before making it. Use when the user asks who is in their Lighthouse organization, or to invite, remove or change the role of a member, manage business groups, or change organization settings.
---

# Manage members, groups and settings

These tools are for organization admins only, and every call checks it. Each write changes what real people can do, or sends them an email, so read first, show the exact change, and wait for a yes before each write.

If a tool named here is missing, the user's key or role does not allow it. Say which one and that it can be changed in Lighthouse under Settings, API & Agents.

## Read

- Members, roles and pending invitations: `list_organization_members` (set includePendingInvitations to see invitations).
- Find a person's user id by name or email: `search_organization_users`.
- Business groups and their members: `list_business_groups`. Groups decide who sees the metrics and data sources shared with them.
- Organization settings (time zone, error notifications, metric suggestions): `get_organization_settings`.

## Write (one confirmation per change)

- **Invite**: `invite_user` with the email and role (admin or member). It sends a real email that cannot be recalled. Check the address is not already a member or invited, confirm the exact address and role, and call it once.
- **Change a role**: `change_member_role`. Takes effect at once. It cannot change the user's own role or demote the last admin.
- **Remove a member**: `remove_member`. They lose access at once and can only come back by a new invitation; their metrics stay. Confirm the person by name and email first.
- **Create a group**: `create_business_group` with members who already belong to the organization (ids from `search_organization_users`). It does not invite anyone.
- **Change a group**: `update_business_group` with only what changes. Removing a member can take away their access to what is shared with the group; a group keeps at least one member.
- **Resend, revoke an invitation, or delete a group**: not available through MCP yet; do it in the Lighthouse web app under Settings.
- **Settings**: `update_organization_settings` for the time zone, error notifications and metric suggestions. The AI model, error destination, billing and the MCP switch are changed in the Lighthouse web app.

For a list of changes, confirm the whole list, then make them one at a time and stop at the first error.

## Safety rules

- Never write without the admin's yes for that change. A request like "clean up the members" is not a yes to remove anyone in particular; list who you would remove and ask.
- Member lists hold names and email addresses. Show them only to the admin who asked; do not copy them anywhere else.
- A role change can take up to a minute to apply to some tools. If a call says the user is not an admin right after a change, wait and retry once.

## Report back

- Each change made: who, what, and the result.
- Anything refused and why (already invited, last admin, not a member).
- Invitations sent, so the admin knows emails went out.
