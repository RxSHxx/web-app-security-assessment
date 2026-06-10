# ❌ VULNERABLE IMPLEMENTATION
# Trusts the fare value submitted directly from the client
def checkout(request):
    user_provided_fare = request.POST.get('fare')
    return process_payment(user_provided_fare)


# ✅ SECURE IMPLEMENTATION
# Ignores client-supplied fare — fetches authoritative price from database
def checkout(request):
    schedule_id = request.POST.get('schedule_id')
    seat = request.POST.get('seat')
    # Server looks up the real price — client cannot influence this value
    authorized_fare = Database.get_fare_by_schedule(schedule_id, seat)
    return process_payment(authorized_fare)
