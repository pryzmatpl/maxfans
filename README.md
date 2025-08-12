```
for enable in /sys/class/hwmon/hwmon*/pwm*_enable /sys/class/drm/card*/device/hwmon/hwmon*/pwm*_enable; do
    pwm="${enable%_enable}"
    if [ -e "$enable" ]; then
        # Switch to manual mode
        if grep -q manual "$enable" 2>/dev/null; then
            echo manual | sudo tee "$enable"
        else
            echo 1 | sudo tee "$enable"
        fi
        # Now set to max
        if [ -e "${pwm}_max" ]; then
            max=$(cat "${pwm}_max")
            echo "$max" | sudo tee "$pwm"
        else
            echo 255 | sudo tee "$pwm"
        fi
    fi
done


# --- verification / stats ---
echo -e "\n=== Fan control status ==="
for hw in /sys/class/hwmon/hwmon*; do
    [ -e "$hw/name" ] || continue
    name=$(cat "$hw/name" 2>/dev/null || echo "?")
    echo "[$(basename "$hw")] name=$name"

    # list PWM controls
    for pwm in "$hw"/pwm[0-9]; do
        [ -e "$pwm" ] || continue
        en="${pwm}_enable"
        maxf="${pwm}_max"
        val=$(cat "$pwm" 2>/dev/null || echo "?")
        enval=$(cat "$en" 2>/dev/null || echo "?")
        maxv=$([ -e "$maxf" ] && cat "$maxf" || echo "255")
        printf "  %-20s mode=%-3s value=%-4s max=%-4s\n" "$(basename "$pwm")" "$enval" "$val" "$maxv"
    done

    # list RPM sensors for reference
    for fan in "$hw"/fan[0-9]_input; do
        [ -e "$fan" ] || continue
        rpm=$(cat "$fan" 2>/dev/null || echo "?")
        printf "  %-20s rpm=%s\n" "$(basename "$fan")" "$rpm"
    done
done
echo "==========================="
```


