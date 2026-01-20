# Automatic Account Synchronization

This guide explains how to configure automatic synchronization of your accounts with data providers like Plaid, SimpleFin, and others.

## Overview

Automatic sync updates your account data from connected providers on a daily schedule. This ensures your financial data stays current without manual intervention.

### Why Configure Sync Timing?

Different banks and financial institutions update their data at different times throughout the day. By configuring when Sure syncs your accounts, you can:

- Ensure you're getting the freshest data after your bank updates
- Avoid unnecessary sync attempts before data is available
- Reduce API usage by syncing at optimal times

## Configuration

### Accessing Sync Settings

1. Log in to your Sure instance
2. Navigate to **Settings > Hosting**
3. Scroll to the **Sync Settings** section

### Enable Automatic Sync

1. Toggle **Enable automatic sync** to ON
2. The setting saves automatically

### Set Sync Time

1. Ensure automatic sync is enabled
2. Click the **Sync time** field
3. Select your preferred time using the time picker
4. The setting saves automatically

> [!NOTE]
> The sync time uses your timezone from your user preferences. The system automatically converts this to UTC for scheduling.

### Disable Automatic Sync

If you prefer to sync accounts manually:

1. Toggle **Enable automatic sync** to OFF
2. The scheduled sync job is removed immediately

## How It Works

### Technical Details

Automatic sync uses Sidekiq cron jobs to schedule daily synchronization:

- **Job name**: `sync_all_accounts`
- **Queue**: `scheduled`
- **Frequency**: Once per day at your configured time

### Timezone Handling

The system handles timezones automatically:

1. You set the sync time in your local timezone
2. Sure converts this to UTC for the cron schedule
3. The job runs at the correct time regardless of server timezone

### Immediate Effect

Changes to sync settings take effect immediately:

- No server restart required
- The cron job updates as soon as you save
- You can verify the schedule in your Sidekiq dashboard

## Use Cases

### Example 1: Morning Bank Updates

Your bank updates account data at 8:00 AM daily.

**Solution**: Set sync time to 9:00 AM to ensure fresh data is available.

### Example 2: Manual Control

You prefer to sync accounts only when needed.

**Solution**: Disable automatic sync and use the manual sync button in the accounts page.

### Example 3: Multiple Timezones

You travel frequently and change timezones.

**Solution**: Update your timezone in user preferences. The sync time automatically adjusts to your new timezone.

## Troubleshooting

### Sync Doesn't Run at Expected Time

**Check your timezone setting**:

1. Go to **Settings > Profile**
2. Verify your timezone is correct
3. Update if needed and reconfigure sync time

**Verify the sync schedule**:

1. Access your Sidekiq dashboard (typically at `/sidekiq`)
2. Click **Cron** in the sidebar
3. Look for the `sync_all_accounts` job
4. Check the schedule matches your expected time in UTC

### Invalid Time Format Error

If you see an error about invalid time format:

1. Ensure you're using the time picker (don't type manually)
2. The format should be HH:MM (24-hour format)
3. If the error persists, try disabling and re-enabling automatic sync

### Sync Job Not Created

If the cron job doesn't appear in Sidekiq:

1. Check the Rails logs for errors
2. Verify Sidekiq is running (`docker compose ps` should show the worker container)
3. Try toggling automatic sync off and on again
4. Restart the worker container if needed:

```bash
docker compose restart worker
```

### Accounts Not Syncing

If automatic sync is enabled but accounts aren't updating:

1. Check that your provider credentials are valid
2. Look for sync errors in **Settings > Hosting > Sync History**
3. Try a manual sync to identify any connection issues
4. Review worker logs for detailed error messages:

```bash
docker compose logs worker
```

## Related Settings

### Include Pending Transactions

The **Include pending transactions** setting controls whether syncs fetch pending transactions from your providers. This setting works independently of automatic sync timing.

### Manual Sync

You can always trigger a manual sync regardless of automatic sync settings:

1. Go to the **Accounts** page
2. Click the sync button for individual accounts
3. Or use **Sync all** to update all connected accounts

## Best Practices

- **Set sync time after your bank updates**: Most banks update overnight or early morning
- **Monitor sync history**: Check for patterns in sync failures or delays
- **Use manual sync for immediate updates**: Don't wait for the scheduled sync if you need current data
- **Keep provider credentials current**: Expired credentials will cause sync failures

## Environment Variables

For advanced users, you can configure sync settings via environment variables:

```bash
# In your .env file
AUTO_SYNC_ENABLED=true
AUTO_SYNC_TIME=09:00
AUTO_SYNC_TIMEZONE=America/New_York
```

> [!WARNING]
> Environment variables override UI settings. If you set these variables, the UI controls will be disabled.
