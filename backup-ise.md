# Backup ISE Configuration

Execute the ISE configuration backup automation package:

1. Navigate to `ise_backup_automation/` directory
2. Activate the shared virtual environment
3. Run the `ise_backup.py` script
4. Monitor the backup progress
5. Verify the backup file on VM (10.122.3.205)
6. Provide a detailed summary report

The automation will:
- Connect to ISE server (https://10.122.59.60)
- Login with admin credentials
- Navigate to Administration > Backup & Restore
- Create Configuration Data Backup named "ISE60"
- Use repository "gs-ubuntu-sche-vm1"
- Encrypt with key "GScxlab123"
- Wait for completion (typically 5-7 minutes)
- Verify backup file exists on VM with correct date tag

Report should include:
- Backup filename (format: ISE60-CFG10-YYMMDD-HHMM.tar.gpg)
- File size (~500M)
- Location on VM
- Completion status
- File naming breakdown explanation
